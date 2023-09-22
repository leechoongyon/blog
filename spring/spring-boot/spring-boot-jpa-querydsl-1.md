## 1. spring boot, jpa, querydsl 정리 - 1편 환경 구성

## 2. 환경 
- Intellij
- Spring boot
- jpa
- querydsl
- gradle (multi project) 

## 3. github source
- [spring-boot-jpa-example source](https://github.com/leechoongyon/spring-boot-example/tree/master/spring-boot-jpa-example) 참고

## 4. 프로젝트 전체적인 구성
- RootProject : spring-boot-example
- SubProject : spring-boot-jpa-example

## 5. 프로젝트 구성 방법
- [Intellij multi project 만드는 법](https://leechoongyon.github.io/posts/intellij-gradle-multi-project) 참고
- root build.gradle
```groovy
plugins {
    id 'org.springframework.boot' version '2.3.1.RELEASE'
    id 'io.spring.dependency-management' version '1.0.9.RELEASE'
    id 'java'
}

bootJar.enabled = false

allprojects {
    apply plugin: 'java'
    apply plugin: 'io.spring.dependency-management'
    apply plugin: 'org.springframework.boot'

    group = 'org.example'
    version = '1.0.0'
    sourceCompatibility = '1.8'

    configurations {
        compileOnly {
            extendsFrom annotationProcessor
        }
    }

    repositories {
        mavenCentral()
    }

    dependencies {
        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
        annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"

        testImplementation('org.springframework.boot:spring-boot-starter-test') {
            exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
        }
        runtimeOnly 'com.h2database:h2'

        testCompile group: 'junit', name: 'junit', version: '4.12'
    }

    test {
        useJUnitPlatform()
    }
}



project(':spring-boot-batch-example') {
    bootJar.enabled = true
    jar.enabled = true
}

project(':spring-boot-jpa-example') {
    bootJar.enabled = true
    jar.enabled = true
}

```


- jpa sub build.gradle

```groovy
plugins {
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

apply plugin: "com.ewerk.gradle.plugins.querydsl"

group 'org.example.jpa'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

bootJar {
    mainClassName = 'org.example.jpa.JpaExampleApplication'
}

dependencies {
    implementation("com.querydsl:querydsl-jpa") // querydsl
    implementation("com.querydsl:querydsl-apt") // querydsl
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
}

def querydslDir = "$buildDir/generated/querydsl"

querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
sourceSets {
    main.java.srcDir querydslDir
}
configurations {
    querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
```


