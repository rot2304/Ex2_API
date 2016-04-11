# דו"ח מסכם- קובץ מזג האוויר

##תיאור הנתונים- מידע על הקובץ:
* מקובץ הטרנזקציות של הנדל"ן בסקרמנטו הוא רשימה של 985 טרנזקציות נדל"ן באיזור סקרמנטו שדווחו לאורך תקופה של חמישה ימים כפי שדווח ב Sacramento Bee
* לקובץ יש מידע על הכתובות שניתן וגם מיקומי מיקום (קוארדינטות אורך ורוחב).
* הקובץ מכיל את העמודות הבאות:
רחוב, עיר, מיקוד, מדינה, חדרי שינה, מקלחות, מטר מרובע, סוג, תאריך המכירה, מחיר, קו אורך, קו רוחב


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

