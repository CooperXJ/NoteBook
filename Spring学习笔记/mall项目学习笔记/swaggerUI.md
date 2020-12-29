使用步骤：

​	1. 导入依赖

```xml
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-annotations</artifactId>
    <version>1.6.0</version>
</dependency>

<!--Swagger-UI API文档生产工具-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.7.0</version>
</dependency>
```

2. 常用注解解析

   - @Api：用于修饰Controller类，生成Controller相关文档信息
   - @ApiOperation：用于修饰Controller类中的方法，生成接口方法相关文档信息
   - @ApiParam：用于修饰接口中的参数，生成接口参数相关文档信息
   - @ApiModelProperty：用于修饰实体类的属性，当实体类是请求参数或返回结果时，直接生成相关文档信息

3. swagger-UI文件

   ```java
   /**
    * Swagger2API文档的配置
    */
   @Configuration
   @EnableSwagger2
   public class Swagger2Config {
       @Bean
       public Docket createRestApi(){
           return new Docket(DocumentationType.SWAGGER_2)
                   .apiInfo(apiInfo())
                   .select()
                   //为当前包下controller生成API文档
                   .apis(RequestHandlerSelectors.basePackage("com.cooper.controller"))
                   //为有@Api注解的Controller生成API文档
   //                .apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
                   //为有@ApiOperation注解的方法生成API文档
   //                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                   .paths(PathSelectors.any())
                   .build();
       }
   
       private ApiInfo apiInfo() {
           return new ApiInfoBuilder()
                   .title("SwaggerUI演示")
                   .description("mall-tiny")
                   .contact("macro")
                   .version("1.0")
                   .build();
       }
   }
   ```

4. 在需要描述的方法名上添加swagger注释@ApiOperation（“xxxxx”）

   ```java
   @ApiOperation("获取所有品牌列表")
   @RequestMapping(value = "listAll", method = RequestMethod.GET)
   @ResponseBody
   public CommonResult<List<PmsBrand>> getBrandList() {
       return CommonResult.success(brandService.listAllBrand());
   }
   ```

   描述的参数方法名添加swagger注释@ApiParam

   ```java
   @ApiOperation("分页查询品牌列表")
   @RequestMapping(value = "/list", method = RequestMethod.GET)
   @ResponseBody
   public CommonResult<CommonPage<PmsBrand>> listBrand(@RequestParam(value = "pageNum", defaultValue = "1")
                                                       @ApiParam("页码") Integer pageNum,
                                                       @RequestParam(value = "pageSize", defaultValue = "3")
                                                       @ApiParam("每页数量") Integer pageSize) {
       List<PmsBrand> brandList = brandService.listBrand(pageNum, pageSize);
       return CommonResult.success(CommonPage.restPage(brandList));
   ```

5. 给实体类添加swagger注释

   在实体类的字段上添加@ApiModelProperty(value = "xxxx")

   ```java
   @ApiModelProperty(value = "产品数量")
   ```

