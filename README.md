# דו"ח מסכם- קובץ מזג האוויר

##תיאור הנתונים- מידע על הקובץ:
* מקובץ הטרנזקציות של הנדל"ן בסקרמנטו הוא רשימה של 985 טרנזקציות נדל"ן באיזור סקרמנטו שדווחו לאורך תקופה של חמישה ימים כפי שדווח ב Sacramento Bee
* לקובץ יש מידע על הכתובות שניתן וגם מיקומי מיקום (קוארדינטות אורך ורוחב).
* הקובץ מכיל את העמודות הבאות:
רחוב, עיר, מיקוד, מדינה, חדרי שינה, מקלחות, מטר מרובע, סוג, תאריך המכירה, מחיר, קו אורך, קו רוחב


## תיעוד הקוד -קבלת סט המידע מה-API:

```Install.packages(“rwunderground”)
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
```saveRDS(dataset, file="weather_data.Rda")
dataset <- readRDS(file="weather_data.Rda")
```

