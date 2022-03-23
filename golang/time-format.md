# 时间序列化

## 重写序列化方式

基于MarshalJSON、UnmarshalJSON 重写

```
// date format
const timeFormat  = "2006-01-02T15:04:05.000"
const dateFormat  = "2006-01-02"

type JsonTime time.Time

func (t JsonTime) MarshalJSON() ([]byte, error)  {
	var stamp = fmt.Sprintf("\"%s\"", time.Time(t).Format(timeFormat))
	return []byte(stamp), nil
}

func (t *JsonTime) UnmarshalJSON (data []byte) error {

	date := string(data)
	if len(date) == 10 {
		tmp, _ := time.ParseInLocation(dateFormat, date, time.Local)
		*t = JsonTime(tmp)
	} else {
		tmp, _ := time.ParseInLocation(timeFormat, date, time.Local)
		*t = JsonTime(tmp)
	}

	return nil
}
```

## 日期格式化
6-1-2-3-4-5原则 2006-01-02T15:04:05.000