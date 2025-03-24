# [File Upload Using Feign - multipart/form-data](https://stackoverflow.com/questions/31752779/file-upload-using-feign-multipart-form-data)

For spring boot 2 and **spring-cloud-starter-openfeign** use this code:

```java
@PostMapping(value="/upload", consumes = "multipart/form-data" )
QtiPackageBasicInfo upload(@RequestPart("package") MultipartFile package);
```

You need to change @RequestParam to @RequestPart in the feign client call to make it work, and also add consumes to the @PostMapping.