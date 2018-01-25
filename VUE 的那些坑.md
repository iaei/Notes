# VUE 的那些坑

## 表单自动跳转

vue中表单自动跳转会导致页面刷新

解决方法 `<form onsubmit="return false"></form>`

