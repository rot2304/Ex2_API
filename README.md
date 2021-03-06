# דו"ח מסכם- קובץ מזג האוויר


##תיאור הנתונים- מידע על הקובץ:
מספק לנו כלי עבור קבלת מידע היסטורי על מזג האוויר ותחזיות.
המידע ההיסטורי על מזג האוויר והתחזית כולל את המאפיינים הבאים:
* טמפרטורה
* לחות
* מהירות רוח
* מדד חום
* נקודות טל

בנוסף לכך, הוא כולל גם מידע על הזריחה והשקיעה, תנאי גאות ושפל, דמוי לווין והתרעות על מזג אוויר והוריקן.

## תיעוד הקוד -קבלת סט המידע מה-API:

```{r}
Install.packages(“rwunderground”)
Require(rwunderground)
location<- “Israel/Jerusalem”
dataset<-forecast10day(location)
dataset[“Area”]<- location
dataset$“Land”[Area==”Israel/Jerusalem”]<-“Europe”
location<-“Italy/Rome”
temp<-forecast10day(location)
dataset[“Area”]<-location 
dataset$“Land”[Area==” Italy/Rome”]<-“Europe”
dataset<- rbind(temp,dataset)

```

ע"מ ליצור סט נתונים עשיר מספיק כך שנוכל לנתחו ולהעו מסקנות מבוססות ומגובשות ביצענו פעולה זאת לעוד מספר רב של ערים במדינות שונות ובכך אנו בעצם רואים את התחזית לעשרת הימים הקרובים לכל עיר

בנוסף לכך הוספנו עוד שתי עמודות לסט הנתונים:

1.יבשת

2.עיר

זאת ע"מ שנוכל להסיק מסקנות יותר מבוססות לגבי הקשר בין התחזית לבין היבשת והעיר.

## שמירת סט המידע וטעינתו לתוך DataFrame:


```{r}
saveRDS(dataset, file="weather_data.Rda")
dataset <- readRDS(file="weather_data.Rda")
```
##ניתוח הנתונים:
###גרפים:
#### boxplot:

```{r}
par(mar=c(9.5,3,0.5, 0.3))
boxplot(dataset$temp_high~dataset$Area,col="blue",las=2 ) 
```
![](https://cloud.githubusercontent.com/assets/17852872/14420472/e548c622-ffd4-11e5-8b02-eb71f6c49600.png)

בגרף זה ניתן לראות את השוני בטווחי מזג האוויר בערים השונות. ניתן לראות שמזג האוויר בערים השונות הוא די שונה וכל עיר טווח משלה ואין כ"כ חפיפה במזג האוויר בין הערים.
בגרפים הבאים שניצור ננסה להבין ממה נובע שוני זה ומה משפיע על מזג האוויר בערים השונות, אם ישנו קשר בין היבשות שבו הם נמצאות.

#### ScatterPlot:
```{r}
dataset$Area<- as.factor(dataset$Area)
dataset$temp_high<- as.integer(dataset$temp_high)
dataset$Land <- as.factor(dataset$Land)
par(mar=c(10,4,4,2) + 0.1)
install.packages("reshape")
require(“reshape”)
ggplot(data=dataset, aes(x=Area, y=temp_high,col=Land)) +
+     geom_line(size=2) +  theme(axis.text.x=element_text(angle=90,hjust=1,vjust=0.5))
```

![](https://cloud.githubusercontent.com/assets/17852872/14420473/e549db02-ffd4-11e5-84c8-7a0d0addc4fc.png)

בגרף זה טווח המזג אוויר (המשוקלל ע"י טמפטורה ורוחות) אשר של כל הערים אשר באותה היבשת נצבעו באותו הצבע. ניתן לראות שטווח מזגי האוויר של ערים אשר באותה בישת (באותו צבע בגרף) מתקבצים סביב אותו איזור ונמצאים פחות או יותר באותו טווח. מכאן אנחנו יכולים להסיק מסקנה שישנו קשר ישיר מובהק בין היבשת למזג האוויר, כלומר היבשת בה נמצאת העיר משפיעה באופן ישירה על מזג האוויר בה ולכן בערים באותה היבשת ישנו מזג אוויר דומה.

#### ניתוח חשיבות מאפיינים שונים:
```{r}
dataset <- readRDS(file="weather_data.Rda")
dataset = within(dataset, {cat_temp = ifelse(temp_high > 73, 1, 0)})
write.csv(dataset, "test.csv", row.names=FALSE)
dataset<-read.csv("./test.csv")
set.seed(nrow(dataset))
control <- trainControl(method="repeatedcv", number=10, repeats=3)
dataset$cat_temp <-as.factor(dataset$cat_temp)
dataset$Area <-as.numeric(dataset$Area)
dataset$Land <-as.numeric(dataset$Land)
x<- dataset[,c("Land","Area")]
y<- dataset$cat_temp
model <- train(x, y, method="lvq", preProcess="scale", trControl=control)
importance <- varImp(model, scale=FALSE)
plot(importance)
```
![](https://cloud.githubusercontent.com/assets/17852872/14420471/e548a7be-ffd4-11e5-82fc-946b6fa69143.png)


לצורך ביצוע ניתוח חשיבות המאפיינים הוספנו עמודה  קטגוריאלית של טמפרטורה (הגדרנו כי מעל 73 מעלות נחשב חם ומתחת לכך נחשב קר) ובדקנו עמודה זאת ע"פ איזור גיאורפי (עיר+מדינה) ובנוסף ע"פ היבשת בה היא נמצאת ע"מ לראות לאיזה ממאפיינים אלה יש השפעה יותר חזקה על הטמפרטורה.
מצאנו ע"פ תוצאות הגרף כי הטמפרטורה איננה נקבעת ע"פ עיר בלבד, כי אם לפי היבשת בה היא נמצאת. כלומר באיזורים אשר שוכנים באותה היבשת טמפרטורות דומות. מכך וכן מהשלבים הקודמים התגבשה המסקנה הסופית אשר לפיה ליבשת ישנה השפעה מכרעת על המזג אוויר וישנו קשר מובהק בין היבשת בה שוכנת העיר לבין טווח המזג אוויר שבה ועל כן ערים אשר באותה היבשת להם טווח מזג האוויר דומה.

##	סיכום, מסקנות והמלצות להמשך מחקר:
כמו שהראינו ישנו קשר ישיר בין מזג האוויר בעיר מסוימת לבין היבשת שבה היבשת נמצאת.
הסקנו כי ניתן לשער את המזג האוויר בעיר מסוימת ולחזות אותו ע"פ היבשת בה נמצאת אותה העיר כי ערים אשר באותו יבשת חולקים טווח דומה של מזג האוויר.
המקור נתונים שלנו מפיק רק חזית לעשרה ימים הקרובים. על כן להמשך מחקר וקבלת מסקנות יותר מוחשיות נסה לבצע מחקר על סט נתונים אשר נותן תחזית לתקופות יותר ארוכות, או לחלופין על מידע היסטורי לתקופות שונות במהלך השנה ובשנים שונות. בדרך זו נוכל לבסס מסקנות יותר מוחשיות לגבי הקשר בין יבשת למזג אוויר.
