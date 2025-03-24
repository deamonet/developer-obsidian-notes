```text
response.setCharacterEncoding("UTF-8");
response.setContentType("application/json; charset=utf-8");
response.setHeader("Access-Control-Allow-Credentials","true");
response.setHeader("Access-Control-Allow-Origin", req.getHeader("Origin"));
response.setHeader("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
response.setHeader("Access-Control-Allow-Methods", "GET, HEAD, POST, PUT, DELETE, TRACE, OPTIONS, PATCH");
```
