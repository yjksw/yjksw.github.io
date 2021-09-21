---
emoji: ✊
title: Java IOStream 과 파일 입출력 
date: '2021-08-26 23:00:00'
author: 코다
tags: 자바
categories: 자바
---

# 자바 I/O 스트림

- Stream은 데이터의 연속이다. Sequence of Data
- 다르게 말하면 **Stream 이란 한쪽으로 흐르는 통로같은 것이다.** 자바에서 Stream이란 한쪽 source에서 destination으로 흐르는 **데이터를 위한 단방향 통로**이다. 자바에서는 여러 매체를 읽거나 쓸 수 있고 각자를 위한 I/O Stream이 구현되어 있다. (disk files, devices, programs, memory arrays)
- Stream은 단방향 통신이기때문에 들어오는 데이터, 나가는 데이터에 따로 InputStream, OutputStream이 있는 것이다. 
- I/O 스트림은 여러가지 종류의 데이터들을 처리할 수 있다: 바이트, primitive data type, characters, objects
- Stream은 단순히 데이터를 전달하는 역할만 하기도하고, 몇몇 stream은 데이터를 조작하고 편리하게 변환하는 역할을 수행하기도 한다.
- 모든 Stream은 사용 후 반드시 `close()` 해주어야 한다. (`try-with-resources`를 추천한다.)
<p align="center"><img width=400 src="https://user-images.githubusercontent.com/63405904/130890827-a907eff4-5edc-437c-b8d3-50cdb2cac0c9.png"><br>TCP School</p>

<br>

## 바이트 기반 Stream 
- 바이트를 기반으로 입출력하는 스트림이다. 
- FileInputStream(파일), ByteArrayInputStream(메모리), PipeInputStream(프로세스), AudioInputStream(오디오 장치) 등등이 있다. 

<br>

## 보조 Stream
- 실제로 데이터를 주고받는 스트림이 아니라, 다른 스트림의 기능을 추가적으로 보조하여 새로운 기능을 수행할 수 있도록 해주는 스트림이다. 
- 생성시 인자로 `InputStream`, `OutputStream` 등을 받기도 한다. 
- 몇가지 예시)
    1. BufferedInputStream / BufferedOutputStream - 버퍼를 이용한 입출력으로 데이터 처리 속도를 높일 수 있다. 
        - 지정되지 않았을 때 기본 버퍼 사이즈는 `8192` 이다.
    2. DataInputStream / DataOutputStream - 자바의 기본 타입으로 데이터를 읽음
    3. 등등

<br>

## 문자 기반 Stream
- Reader, Writer 를 통해서 UTF-8 등등의 인코딩 텍스트를 처리할 수 있다. 
    - 이것은 필터의 일종이다. 
- 이것을 사용하면 데이터를 byte 단위가 아닌, char 단위로 처리할 수 있다. 
- 대표적으로 자주 사용하는 것이 `BufferedReader`가 있다. 

```java
InputStream inputStream = new ByteArrayInputStream(content.getByte());
BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
```
- 또 많이 사용되는 것으로는 `FileReader`, `CharArrayReader`, `PipedReader`, `StringReader` 등등이 있다. (각각 Writer도 있다.)

<br>
<br>

# InputStream
- InputStream 은 *프로그램 입장*에서 데이터를 읽어드리는 것이다. 
    
    <p align="center"><img width=400 src="https://user-images.githubusercontent.com/63405904/130889921-1ef178ae-5fa1-4597-a173-ffa3165cf897.png"><br>이미지 출처: 오라클 docs</p>
- InputStream은 입출력 스트림의 기본 클래스이며 `read()` 메소드를 추상메소드를 지원한다. 
- `read()`메소드를 적절하게 구현하여 사용허던지, 이미 구현한 하위 기타 InputStream을 활용하면 된다. 
- 기본적으로 바이트를 읽어드린다. `read(byte[], int off, int len)`, `read(byte[])`

### 바이트를 읽는다는데 왜 반환값은 int일까?
- 보면 `InputStream`의 overloading 되어 있는 모든 `read()` 메소드의 반환값이 `int` 이다. 
- 그 이유는 스트림에 있는 데이터를 모두 읽었을 때 `-1` 을 반환하는데 반환값이 byte이면 `-1` 을 반환할 수 없기 때문이다. 
- 데이터를 읽을 때 우선 0-255 사이의 값을 int로 반환하고, 해당 값을 -128 부터 127 사이의 byte 타입으로 변환한다. 

<br>

## InputStream to String
- 여러가지 방법이 있겠지만 가장 간단한 방법은 다음이다. 
- InputStream에서 읽은 byte 데이터를 `new String()`을 통해 반환한다. 
- 이때 두번째 인자로 인코딩 타입도 정할 수 있다. 
```java
final String data = 
    new String(inputStream.readAllBytes(), StandardCharsets.UTF_8);
```

<br>
<br>

# OutputStream
- 다른 매체 (콘솔, 모니터, 파일 등등)에 데이터를 쓸 때 사용되는 출력 스트림이다. 
        <p align="center"><img width=400 src="https://user-images.githubusercontent.com/63405904/130900408-a052d0ba-0ac3-4818-8fc7-80c17f156d41.png"><br>이미지 출처: 오라클 docs</p>
- 그냥 `write()`는 바이트 단위로 쓰기 때문에 비효율적이다. `read()`와 마찬가지로 `write()`도 byte[]을 인자로 넣어서 1 바이트 이상을 효율적으로 쓸 수 있다. 
   ```java
   write(byte[]);
   write(byte[], int off, int len);
   ```
<br>

## `ByteArrayOutputStream` vs. `BufferedOutputStream`
- `ByteArrayOutputStream`는 메모리에 데이터를 출력하는 스트림이다. 
- `BufferedOutputStream`은 버퍼링 된 출력 스트림을 생성하는 filter stream의 일종으로 생성시 인자로 OutputStream을 받는다. 
    
    ```java
    BufferedOutputStream stream = new BufferedOutputStream(outputStream);
    ```
    - `BufferedOutputStream`이 직접 출력 스트림에 쓰는 것이 아니라 우선 버퍼에 써서 저장이 되도록 지원하는 것이다. 그렇기 때문에 인자로 출력 스트림을 받아야한다. 
    - 버퍼의 사이즈는 지정할 수 있으며 default 버퍼 사이즈는 `8192`이다.
    - `BufferedOutputStream`의 `close()`가 호출되면 버퍼에 있는 내용이 출력 스트림에 쓰인다. 
    - 스트림을 닫지 않은 채로 내용을 쓰고 싶다면 `flush()`를 호출하면 된다.  

### 차이점
- `ByteArrayOutputStream`은 unbuffered I/O이고, `BufferedOutputStream`은 buffered I/O 이다. 
<br>

### 왜 buffered I/O를 사용할까?
- unbuffered I/O를 사용한다는 것은 매번 입력 및 출력시 OS에 직접적인 요청이 간다는 것이다. 디스크 접근, 네트워크, 등등의 OS관련 요청이 많아지면서 그 비용으로 인해 프로그램의 효율이 배우 떨어진다. 
- 그것을 해결하기 위해서 자바가 buffered I/O를 지원하도록 했다. 
    - Buffered input stream은 데이터를 buffer라는 메모리 공간에서 읽고 native input API 는 버퍼가 비었을때만 호출된다.   
    - Buffered output stream은 반대로 출력하고자 하는 데이터가 buffer에 가득 찼을 때 native output API가 호출된다.
- 프로그램은 unbuffered I/O인 입출력 스트림을 buffered 스트림 선언시 인자로 넘겨 버퍼링 되도록 한다. 
- 바이트 버퍼링을 위해 `BufferedInputStream`, `BufferedOutputStream`이 있다.
- 문자 버퍼링을 위해 `BufferedReader`, `BufferedWriter`가 있다. 
<br>

### `flush()`는 언제?
- 언제 buffer를 비우는지가 중요한 포인트이다. 버퍼를 비우고 target에 입출력을 반영하는 것을 flushing이라고 한다. 
- 어떤 buffered output class들은 autoflush를 지원해서 특정 이벤트 동작시 자동 flush가 되도록 한다. 
    - 예를 들어 `PrintWriter`가 `println`, `format` 등을 기점으로 `flush`를 한다. 
- 버퍼에 있는 내용을 반영하고 싶을 때 수동으로 `flush()`를 호출하면 된다. 참고로 `flush()`는 모든 스트림에 존재하는 메소드이지만 버퍼링을 지원하는 스트림이 아니면 아무 효과가 없다. 

<br>
<br>
<br>

# File
## path, absolute path, canonical path
- 파일 이름으로 경로를 찾아 해당 파일을 읽는 실습을 하면서 여러 종류의 path가 등장한다. 
1. path - 가장 기본적인 path로 주로 입력된 경로 그대로이다. 
2. absolute path - 처음 root 부터 절대 경로를 나타낸다. 
    - solaris의 경우 `/home/sally/statusReport` 이런 형태를 지닌다. 
    - Windows의 경우 `C:\home\sally\statusReport` 이런 형태를 지닌다. 
    - 루트 디렉토리부터 시작하면 절대경로이다. 그러니까 다음 두가지 경우도 모두 절대경로이다. 
    ```
    /home/../home/originfile
    /home/./././originfile
    ```
3. canonical path - 따라서 절대경로이면서 단 하나의 형식으로 유니크하게 표현하는 것이 canoncial path이다. 즉 위의 절대경로를 canonical path로 표현하면 다음과 같이 된다. 모든 파일의 canoncial path는 단 하나이며 같은 리소스일 경우 그 생김새는 항상 같다. 
    ```
     /home/originfile
    ```
<br>

## Java에서 리소스 파일 읽기
- Java에서 리소스 파일을 읽을 때는 다음과 같이 하면 된다. 
    ```java
    URL resource = getResource().getClassLoader().getResource(fileName);
    Path path = new File(resource.getPath()).toPath();
    List<String> content = Files.readlAllLines(path);
    ```
- 하지만 위와 같이 실행하여 IDE에서 제대로 돌아가더라도 jar로 실행을 시켜보면 리소스를 찾지 못한다는 오류가 발생한다. jar 내부에서 되어있는 파일 구조가 조금 다르기 때문이다. 
- 찾은 해결책은 resource를 inputStream으로 받아서 읽는 것이다. 
    ```java
    InputStream in = getClass().getResourceAsStream("/" + fileName);
    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(in));

    String content;
    final List<String> actual = new ArrayList<>();

    while((content = bufferedReader.readLine()) != null) {
        actual.add(content);
    }
    ```
    - 아직 테스트는 해보지 않았으나 조만한 해볼 예정이다. 

<br>
<br>
<br>

**[참고링크]**
- [https://docs.oracle.com/javase/tutorial/essential/io/streams.html](https://docs.oracle.com/javase/tutorial/essential/io/streams.html)
- [http://tcpschool.com/java/java_io_stream](http://tcpschool.com/java/java_io_stream)
- [https://sgdev.tistory.com/23](https://sgdev.tistory.com/23)
- [https://docs.oracle.com/javase/tutorial/essential/io/buffers.html](https://docs.oracle.com/javase/tutorial/essential/io/buffers.html)
- [https://www.benjaminlog.com/entry/absolute-path-vs-canonical-path](https://www.benjaminlog.com/entry/absolute-path-vs-canonical-path)
- [https://docs.oracle.com/javase/tutorial/essential/io/path.html](https://docs.oracle.com/javase/tutorial/essential/io/path.html)
- [http://daplus.net/java-jar-내에서-리소스-파일-읽기/](http://daplus.net/java-jar-%EB%82%B4%EC%97%90%EC%84%9C-%EB%A6%AC%EC%86%8C%EC%8A%A4-%ED%8C%8C%EC%9D%BC-%EC%9D%BD%EA%B8%B0/)

```toc
```