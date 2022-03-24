# 时间序列化

## 重写序列化方式

基于MarshalJSON、UnmarshalJSON 重写

```shell script
// date format
const timeFormat  = "2006-01-02T15:04:05.000"
const dateFormat  = "2006-01-02"

type JsonTime time.Time

func (t JsonTime) MarshalJSON() ([]byte, error)  {
	var stamp = fmt.Sprintf("\"%s\"", time.Time(t).Format(timeFormat))
	return []byte(stamp), nil
}

func (t *JsonTime) UnmarshalJSON (data []byte) error {

	dateStr := string(data)
	if len(dateStr) == 12 {
		date, _ := time.ParseInLocation(`"` + dateFormat + `"`, dateStr, time.Local)
		*t = JsonTime(date)
	} else {
		dateTime, _ := time.ParseInLocation(`"` + timeFormat + `"`, dateStr, time.Local)
		*t = JsonTime(dateTime)
	}

	return nil
}
```

## 日期格式化
注意点1：  
dateStr := string(data) 出来的时间格式是 
```
""2006-01-02T15:04:05.000""
```
多出双引号，导致 format前需要加 `"`  
注意点2：  
6-1-2-3-4-5原则，   ParseInLocation入参是layout不是format，针对java程序员来说真的很别扭  
2006-01-02T15:04:05.000