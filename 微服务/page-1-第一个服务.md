实现一个传入字符串返回字符串信息,第一个服务只返回字符串本身

网络协议 http + json

请求地址 http://localhost:8080/info

请求参数
```json
{
    string:''
}
```

返回信息
```json
{
    ErrCode:0,
    ErrMsg:'',
    Data:{
        string:''
    },
}
```

# spring boot 版本

maven 配置文件 pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.360.ljw</groupId>
    <artifactId>springboot</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>springboot</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

main -> java 新建 Package Test

新建 TestApplication
```java
package Test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
}

```

新建InfoController
```java
package Test;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import java.util.HashMap;

@RestController
public class InfoController {
    @RequestMapping("/info")
    public HashMap index(HttpServletRequest request) {
        String str = request.getParameter("string");

        HashMap response = new HashMap();
        response.put("ErrCode", 0);
        response.put("ErrMsg", "success");

        HashMap data = new HashMap();
        data.put("String",str);
        response.put("Data", data);

        return response;
    }
}

```
执行
```bash
mvn spring-boot:run
```
访问 http://localhost:8080/info?string=a

返回结果
```json
{
    "ErrMsg":"success",
    "Data":{
        "String":"a"
    },
    "ErrCode":0
}
```

## 学习笔记
tomcat一直启动失败，8080端口被占用
```
netstat –anp | grep 8080
```

# go-kit 版本
新建 test.go
```golang
package main

import (
	"context"
	"encoding/json"
	httptransport "github.com/go-kit/kit/transport/http"
	"github.com/pkg/errors"
	"log"
	"net/http"
)

func InfoEndpoint(_ context.Context, request interface{}) (interface{}, error) {
	req := request.(InfoRequest)

	return InfoResponse{1, "", req.Str}, nil
}

type InfoRequest struct {
	Str string
}

type InfoResponse struct {
	ErrCode int    `json:"errCode"`
	ErrMsg  string `json:"errMsg"`
	Data    string `json:"data"`
}

func decodeRequest(_ context.Context, r *http.Request) (interface{}, error) {
	var request InfoRequest
	r.ParseForm()

	if len(r.Form["string"]) > 0 {
		request.Str = r.Form["string"][0]
		return request, nil
	} else {
		return request, errors.New("string not exists")
	}
}

func encodeResponse(_ context.Context, w http.ResponseWriter, response interface{}) error {
	return json.NewEncoder(w).Encode(response)
}

func main() {
	http.Handle("/info", httptransport.NewServer(InfoEndpoint, decodeRequest, encodeResponse))
	log.Fatal(http.ListenAndServe(":8080", nil))
}

```
## 学习笔记
主要分为3部分
- endpont 服务节点，接收decode之后的request，返回 response
- request 包含 request 内容 及request 的 decode函数
- response 包含 response 结果及 response  的 encode 函数
