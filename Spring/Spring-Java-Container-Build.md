# Spring Gradle 파일에서 Java 도커 컨테이너 빌드하기

```kts
val imageName: String? = System.getProperty("imageName")
val imageTag: String? = System.getProperty("imageTag")

jib {
    from {
        image = "amazoncorretto:17-alpine3.17-jdk"
    }
    to {
        image = imageName
        tags = setOf(imageTag)
    }
    container {
        creationTime = "USE_CURRENT_TIMESTAMP"
        environment = mapOf("TZ" to "UTC")
        jvmFlags = emptyList() //JAVA_TOOL_OPTIONS
        ports = listOf("8080")
    }
}
```

```shell
#!/bin/bash
moduleName=$1
imageName=$2
imageTag=$3
echo "==== SwingBike 빌드 시작 ===="
echo "대상 모듈 이름: $moduleName"
echo "대상 이미지 이름: $imageName"
echo "대상 이미지 태그: $imageTag"
./gradlew :"$moduleName":jib -DimageName="$imageName" -DimageTag="$imageTag"
```
