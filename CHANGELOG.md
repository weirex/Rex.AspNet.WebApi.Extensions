# Change log

## 1.2.1.2（2019.12.25）
- 小调整


## 1.1.8
- Bug修复和小的调整
- WebApiExceptionAttribute 增加自定义错误消息
`config.Filters.Add(new WebApiExceptionAttribute(new ResultModel { Code = -1000, Msg = "服务器开小差了~" }));`
