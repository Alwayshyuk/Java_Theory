# 자바에서의 입출력

## 입출력이란?

I/O란 Input과 Output의 약자로 입력과 출력을 뜻한다.     
입출력은 컴퓨터 내부 또는 외부의 장치와 프로그램간의 데이터를 주고받는 것을 말한다.    

## 스트림(stream)
자바에서 입출력을 수행하려면, 즉 어느 한쪽에서 다른 쪽으로 데이터를 전달하려면,     
두 대상을 연결하고 데이터를 전송할 수 있는 무언가가 필요한데 이것을 스트림stream이라고 정의했다.     

> 스트림이란 데이터를 운반하는데 사용되는 연결통로이다.   

스트림은 연속적인 데이터의 흐름을 물에 비유해서 붙여진 이름인데, 여러 가지로 유사한 점이 많다.    
물이 한쪽 방향으로만 흐르는 것과 같이 스트림은 단방향통신만 가능하기 때문에 하나의 스트림으로 입력과 출력을 동시에 처리할 수 없다.   
그래서 입력과 출력을 동시에 수행하려면 입력을 위한 입력스트림input stream과 출력을 위한 출력스트림output stream이 모두 필요하다.     
스트림은 먼저 보낸 데이터를 먼저 받게 되어 있으며 중간에 건너뜀 없이 연속적으로 데이터를 주고받는다.   
큐Queue와 같은 FIFO구조로 되어있따고 생각하면 된다.   

## 바이트기반 스트림-IputStream, OutputStream

스트림은 바이트 단위로 데이터를 전송하며 입출력 대상에 따라 다음과 같은 입출력스트림이 있다.     

```
입력스트림: FileInputStream
출력스트림: FileOutputStream
입출력 대상의 종류: 파일

입력스트림: ByteArrayInputStream
출력스트림: ByteArrayOutputStream
입출력 대상의 종류: 메모리(byte 배열)

입력스트림: PipedInputStream
출력스트림: PipedOutputStream
입출력 대상의 종류: 프로세스(프로세스간의 통신)

입력스트림: AudioInputStream
출력스트림: AudioOutputStream
입출력 대상의 종류: 오디오장치
```

위와 같이 여러 종류의 입출력 스트림이 있으며, 어떠한 대상에 대해서 작업을 할 것인지     
그리고 입력을 할 것인지 출력을 할 것인지에 따라서 해당 스트림을 선택해서 사용하면 된다.     
예를 들어 어떤 파일의 내용을 읽고자하면 FileInputStream을 사용하면 될 것이다.    
이들은 모두 InputStream 또는 OutputStream의 자손들이며, 각각 읽고 쓰는데 필요한 추상메서드를 자신에 맞게 구현해 놓았다.     
자바에서는 java.io패키지를 통해서 많은 종류의 입출력관련 클래스들을 제공하고 있으며,    
입출력을 처리할 수 있는 표준화된 방법을 제공함으로써 입출력의 대상이 달라져도    
동일한 방법으로 입출력이 가능하기 때문에 프로그래밍을 하기에 편리하다.    

```java
InputStream: abstract int read()
OutputStream: abstract void write(int b)

InputStream: int read(byte[] b)
OutputStream: void write(byte[] b)

InputStream: int read(byte[] b, int off, int len)
OutputStream: void write(byte[] b, int off, int len)

inputStream과 OutputStream에 정의된 읽기와 쓰기를 수행하는 메서드
```
> read의 반환타입이 byte가 아니라 int인 이유는 read()의 반환값의 범위가 0~255와 -1이기 때문이다.    

InputStream의 read()와 OutputStream의 write(int b)는 입출력의 대상에 따라 읽고 쓰는 방법이 다를 것이기 때문에     
각 상황에 알맞게 구현하라는 의미에서 추상메서드로 정의되어 있다.    
read()와 write(int b)를 제외한 나머지 메서드들은 추상메서드가 아니니까 굳이 추상메서드인 read()와 write(int b)를    
구현하지 않아도 이들을 사용할 수 있을 것이라고 생각할 수 있겠지만 사실 추상메서드인 read()와 write(int b)를 이용해서     
구현한 것들이기 때문에 read()와 write(int b)가 구현되어 있지 않으면 이들은 아무런 의미가 없다.     

```java
public abstract class InputStream{
	...
	//입력스트림으로부터 1byte를 읽어서 반환한다. 읽을 수 없으면 -1을 반환한다.
	abstract int read();
	
	//입력스트림으로부터 len개의 byte를 읽어서 byte배열 b의 off위치부터 저장한다.
	int read(byte[] b, int off, int len){
		...
		for(int i = off; i<off+len; i++){
			//read()를 호출해서 데이터를 읽어서 배열을 채운다.
			b[i] = (byte)read();
		}
	...
	//입력스트림으로부터 byte배열 b의 크기만큼 데이터를 읽어서 배열 b에 저장한다.
	int read(byte[] b){
		return read(b, 0, b.length);
	}
	...
}
```

이 코드는 InputStream의 실젲 소스코드의 일부를 이해하기 쉽게 약간 변형한 것인데,   
여기서 read(byte[] b, int off, int len)의 코드를 보면 read()를 호출하는 것을 알 수 있다.     
read()가 추상메서드이지만 이처럼 read(byte[] b, int off, int len)의 내에서 read()를 호출할 수 있다.     
메서드는 선언부만 알고 있어도 호출이 가능하기 때문에, 추상메서드를 호출하는 코드를 작성할 수 있다.    
실제로는 추상클래스를 상속받아서 추상메서드를 구현한 클래스의 인스턴스에 대해서 추상메서드가 호출될 것이기 떄문에    
추상메서드를 호출하는 코드를 작성해도 아무런 문제가 되지 않는다.    

## 보조 스트림

위에서 언급한 스트림 외에도 스트림의 기능을 보완하기 위한 보조스트림이 제공된다.   
보조스트림은 실제 데이터를 주고받는 스트림이 아니기 때문에 데이터를 입출력할 수 있는 기능은 없지만,    
스트림의 기능을 향상시키거나 새로운 기능을 추가할 수 있다.    
그래서 보조스트림만으로는 입출력을 처리할 수 없고, 스트림을 먼저 생성한 다음에 이를 이용해서 보조스트림을 생성해야한다.    
예를 들어 test.txt라는 파일을 읽기위해 FileIntputStream을 사용할 때,     
입력 성능을 향상시키기 위해 버퍼를 사용하는 보조스트림인 BufferedInputStream을 사용하는 코드는 다음과 같다.    

```java
//먼저 기반스트림을 생성한다.
FileInputStream fis = new FileInputStream("text.txt");

//기반스트림을 이용해서 보조스트림을 생성한다.
BufferedInputStream bis = new BufferedInputStream(fis);

//보조스트림인 BufferedInputStream으로부터 데이터를 읽는다.
bis.read();
```

코드 상으로는 보조스트림인 BufferedInputStream이 입력기능을 수행하는 것처럼 보이지만,    
실제 입력기능은 BufferedInputStream과 연결된 FIleInputStream이 수행하고,     
보조스트림인BufferedInputStream은 버퍼만을 제공한다.    
버퍼를 사용한 입출력과 사용하지 않은 입출력간의 성능차이는 상당하기 때문에 대부분의 경우에 버퍼를 이용한 보조스트림을 사용한다.    
BufferedInputStream, DataInputStream, DigestInputStream, LineNumberInputStream,     
PushbackInputStream은 모두 FilterInputStream의 자손들이고,     
FilterInputStream은 InputStream의 자손이라서 결국 모든 보조스트림 역시     
InputStream과 OutputStream의 자손들이므로 입출력방법이 같다.    

```
보조스트림의 종류

입력: FilterInputStream
출력: FilterOutputStream
설명: 필터를 이용한 입출력 처리


입력: BufferedInputStream
출력: BufferedOutputStream
설명: 버퍼를 이용한 입출력 성능향상

입력: DataInputStream
출력: DataOutputStream
설명: int, float과 같은 기본형 단위primary type로 데이터를 처리하는 기능

입력: SequenceInputStream
출력: 없음
설명: 두 개의 스트림을 하나로 연결

입력: LineNumberInputStream
출력: 없음
설명: 읽어 온 데이터의 라인 번호를 카운트(JDK 1.1부터 LineNumberReader로 대체

입력: ObjectInputStream
출력: ObjectOutputStream
설명: 데이터를 객체단위로 읽고 쓰는데 사용, 주로 파일을 이용하며 객체 직렬화와 관련있음

입력: 없음
출력: PrintStream
설명: 버퍼를 이ㅛㅇ하며 추가적인 print 관련 기능 print,printf,println메서드

입력: PushbackInputStream
출력: 없음
설명: 버퍼를 이용해서 읽어 온 데이터를 다시 되돌리는 기능unread, push back to buffer
```

## 문자기반 스트림 - Reader, Writer
Java에서는 한 문자를 의미하는 char형이 1byte가 아니라 2byte이기 때문에      
바이트기반의 스트림으로 2byte인 문자를 처리하는데는 어려움이 있다.      
이 점을 보완하기 위해서 문자기반의 스트림이 제공된다.     
문자데이터를 입출력할 때는 바이트기반 스트림 대신 문자기반 스트림을 사용하자.    

> InputStream - Reader    
> OutputStream - Writer

```
바이트기반 스트림: FileInputStream, FileOutputStream
문자기반 스트림: FileReader, FileWriter

바이트기반 스트림: ByteArrayInputStream, ByteArrayOutputStream
문자기반 스트림: CharArrayInputStream, CharArrayOutputStream

바이트기반 스트림: PipedInputStream, PipedOutputStream
문자기반 스트림: PipedReader, PipedWriter

바이트기반 스트림:StringBufferInputStream(deprecated), StringBufferOutputStream(deprecated) 
문자기반 스트림: StringReader, StringWriter
```

문자기반 스트림의 이름은 바이트기반 스트림의 이름에서 InputStream은 Reader로 OutputStream은 Writer로 바꾸면 된다.     
단, ByteArrayInputStream에 대응하는 문자기반 스트림은 char배열을 사용하는 CharArrayReader이다.    


아래는 바이트기반 스트림과 문자기반 스트림의 읽기와 쓰기에 사용되는 메서드를 비교한 것인데     
byte배열 대신 char배열을 사용한다는 것과 추상메서드가 달라졌다.     
Reader와 Writer에서도 역시 추상메서드가 아닌 메서드들은 추상메서드를 이용해서 작성되었으며,     
프로그래밍적인 관점에서 볼 때 read()를 추상메서드로 하는 것보다 int read(char[] cbuf, int off, int len)를    
추상메서드로 하는 것이 더 바람직하다.    

```
바이트기반 스트림과 문자기반 스트림의 읽고 쓰는 메서드 비교

InputStream
abstract int read()
int read(byte[] b)
int read(byte[] b, int off, int len)

Reader
int read()
int read(char[] cbuf)
abstract int read(char[] cbuf, int off, int len)

OutputStream
abstract void write(int b)
void write(byte[] b)
void write(byte[] b, int off, int len)

Writer
void write(int c)
void write(char[] cbuf)
abstract void write(char[] cbuf, int off, int len)
void write(String str)
void write(String str, int off, int len)
```

보조 스트림 역시 다음과 같은 문자기반 보조스트림이 존재하며 사용목적과 방식은 바이트기반 보조스트림과 다르지 않다.    

```
바이트기반 보조스트림과 문자기반 보조스트림

바이트기반 보조스트림: BufferedInputStream, BufferedOutputStream
문자기반 보조스트림: BufferedReader, BufferedWriter

바이트기반 보조스트림: FilterInputStream, FilterOutputStream
문자기반 보조스트림: FilterReader, FilterWriter

바이트기반 보조스트림: LineNumberInputStream(deprecated)
문자기반 보조스트림: LineNumberReader

바이트기반 보조스트림: PrintStream
문자기반 보조스트림: PrintWriter

바이트기반 보조스트림: PushbackInputStream
문자기반 보조스트림: PushbackReader
```

# 바이트기반 스트림

## InputStream과 OutputStream

InputStream과 OutputStream은 모든 바이트기반의 스트림의 조상이며 아래와 같은 메서드가 선언되어 있다.

```java
InputStream의 메서드

int available()
스트림으로부터 읽어 올 수 있는 데이터의 크기를 반환한다.

void close()
스트림을 닫음으로써 사용하고 있던 자원을 반환한다.

void mark(int readlimit)
현재 위치를 표시해 놓는다. 후에 reset()에 의해서 표시해 놓은 위치로 다시 돌아갈 수 있다.
readlimit은 되돌아갈 수 있는 byte의 수이다.

boolean markSupported()
mark()와 reset()을 지원하는지 알려준다. mark()와 reset()기능을 지원하는 것은 선택적이므로
mark()와 reset()을 사용하기 전에 markSupported()를 호출해서 지원여부를 확인해야한다.

abstract int read()
1 byte를 읽어 온다(0~255). 더 이상 읽어 올 데이터가 없으면 -1을 반환한다.
abstract메서드라서 InputStream의 자손들은 자신의 상황에 알맞게 구현해야한다.

int read(byte[] b)
배열 b의 크기만큼 읽어서 배열을 채우고 읽어 온 데이터의 수를 반환한다.
반환하는 값은 항상 배열의 크기보다 작거나 같다.

int read(byte[] b, int off, int len)
최대 len개의 byte를 읽어서, 배열 b의 지정된 위치off부터 저장한다.
실제로 읽어 올 수 있는 데이터가 len개보다 적을 수 있다.

void reset()
스트림에서의 위치를 마지막으로 mark()가 호출되었던 위치로 되돌린다.

long skip(long n)
스트림에서 주어진 길이n만큼 건너뛴다.
```

```java
OutputStream의 메서드

void close()
입력소스를 닫음으로써 사용하고 있던 자원을 반환한다.

void flush()
스트림의 버퍼에 있는 모든 내용을 출력소스에 쓴다.

abstract void write(int b)
주어진 값을 출력소스에 쓴다.

void write(byte[] b)
주어진 배열b에 저장된 모든 내용을 출력소스에 쓴다.

void write(byte[] b, int off, int len)
주어진 배열b에 저장된 내용 중에서 off번째부터 len개 만큼을 읽어서 출력소스에 쓴다.
```

flush()는 버퍼가 있는 출력스트림의 경우에만 의미가 있으며, OutputStream에 정의된 flush()는 아무런 일도 하지 않는다.   
프로그램이 종료될 떄, 사용하고 닫지 않은 스트림을 JVM이 자동적으로 닫아 주기는 하지만,    
스트림을 사용해서 모든 작업을 마치고 난 후에는 close()를 호출해서 반드시 닫아주어야 한다.   
그러나 ByteArrayInputStream과 같이 메모리를 사용하는 스트림과    
System.in, System.out과 같은 표준 입출력 스트림은 닫아 주지 않아도 된다.

## ByteArrayInputStream과 ByteArrayOutputStream

ByteArrayInputStream/ByteArrayOutputStream은 메모리, 즉 바이트 배열에 데이터를 입출력하는데 사용되는 스트림이다.     
주로 다른 곳에 입출력하기 전에 데이터를 임시로 바이트배열에 담아서 변환 등의 작업을 하는데 사용된다.    

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.util.Arrays;

public class IOEx1 {
	public static void main(String[] args) {
		byte[] inSrc = {0,1,2,3,4,5,6,6,7,8,9};
		byte[] outSrc = null;
		
		ByteArrayInputStream input = new ByteArrayInputStream(inSrc);
		ByteArrayOutputStream output = new ByteArrayOutputStream();
		
		int data = 0;
		
		while((data=input.read())!=-1) {
			output.write(data); 	//void write(int b)
		}
		
		outSrc = output.toByteArray();//스트림의 내용을 byte배열로 반환한다.
		
		System.out.println(Arrays.toString(inSrc));
		System.out.println(Arrays.toString(outSrc));
	}
}
```
ByteArrayInputStream/ByteArrayOutputStream을 이용해서 바이트배열 inSrc의 데이터를 outSrc로 복사하는 예제인데,    
read()와 write()를 사용하는 가장 기본적인 방법을 보여준다.    
while문의 조건식은 아래의 순서로 처리된다.    
1. data=input.read()  //read()를 호출한 반환값을 변수 data에 저장한다.       
2. data != -1  //data에 저장된 값이 -1이 아닌지 비교한다.     
바이트배열은 사용하는 자원이 메모리 밖에 없으므로 가비지컬렉터에 의해 자동적으로 자원을 반환하므로     
close()를 이용해서 스트림을 닫지 않아도 된다.     
read()와 write(int b)를 사용하기 떄문에 한 번에 1byte만 읽고 쓰므로 작업효율이 떨어진다.     
다음 예제는 배열을 사용해서 입출력 작업이 보다 효율적으로 이루어지도록 한다.

```java
import java.io.*;
import java.util.Arrays;
public class IOEx2 {

	public static void main(String[] args) {
		byte[] inSrc = {0,1,2,3,4,5,6,7,8,9};
		byte[] outSrc = null;
		byte[] tmp = new byte[10];
		
		ByteArrayInputStream input = new ByteArrayInputStream(inSrc);
		ByteArrayOutputStream output = new ByteArrayOutputStream();
		
		input.read(tmp, 0, tmp.length);//읽어온 데이터를 배열 tmp에 담는다.
		output.write(tmp, 5, 5);//tmp[5]부터 5개의 데이터를 write한다.
		
		outSrc = output.toByteArray();
		
		System.out.println(Arrays.toString(inSrc));
		System.out.println(Arrays.toString(tmp));
		System.out.println(Arrays.toString(outSrc));
	}
}
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[5, 6, 7, 8, 9]
```

int read(byte[] b, int off, int len)과 void write(byte[] b, int off, int len)를      
사용해서 입출력하는 방법을 보여주는 예제이다.    
이전 예제와 달리 byte배열을 사용해서 한 번에 배열의 크기만큼 읽고 쓸 수 있다.    
바구니tmp를 이용하면 한 번에 더 많은 물건을 옮길 수 있는 것과 같다고 이해하면 좋을 것이다.    
배열을 이용한 입출력은 작업의 효율을 증가시키므로 가능하면 입출력 대상에 다라 알맞은 크기의 배열을 사용하는 것이 좋다.    

```java
import java.io.*;
import java.util.Arrays;
public class IOEx3 {

	public static void main(String[] args) {
		byte[] inSrc = {0,1,2,3,4,5,6,7,8,9};
		byte[] outSrc = null;
		byte[] tmp = new byte[4];
		
		ByteArrayInputStream input = new ByteArrayInputStream(inSrc);
		ByteArrayOutputStream output =  new ByteArrayOutputStream();
		
		try {
			while(input.available()>0) {
				input.read(tmp);
				output.write(tmp);
				
				outSrc = output.toByteArray();
				printArrays(tmp,outSrc);
			}
		} catch (IOException e) {}
	}		
		static void printArrays(byte[] tmp, byte[] outSrc) {
			System.out.println("tmp: "+Arrays.toString(tmp));
			System.out.println("Output Source: "+Arrays.toString(outSrc));
		}
}
tmp: [0, 1, 2, 3]
Output Source: [0, 1, 2, 3]
tmp: [4, 5, 6, 7]
Output Source: [0, 1, 2, 3, 4, 5, 6, 7]
tmp: [8, 9, 6, 7]
Output Source: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 6, 7]
```
read()나 write()가 IOException을 발생시킬 수 있기 때문에 try-catch문으로 감싸주었다.    
available()은 블라킹blocking없이 읽어 올 수 있는 바이트의 수를 반환한다.    
tmp는 보다 나은 성능을 위해 담긴 내용을 지우고 쓰는 것이 아니라 그냥 기존의 내용 위에 덮어 쓴다.    
그래서 tmp의 내용은 [4,5,6,7]에서 8,9를 읽고 난 후 [8,9,6,7]이 된다.   
원하는 결과를 읽기 위해서는 while문을 수정해야 한다.    
원래의 코드는 배열의 내용전체를 출력하지만, 수정된 코드는 읽어온 만큼len만 출력한다.

> 블락킹이란 데이터를 읽어 올 때 데이터를 기다리기 위해 멈춰있는 것을 뜻한다.    
> 예를 들어 사용자가 데이터를 입력하기 전까지 기다리고 있을 때 블락킹 상태에 있다고 한다.

```java
import java.io.*;
import java.util.Arrays;

public class IOEx4 {
	public static void main(String[] args) {
		byte[] inSrc = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
		byte[] outSrc = null;
		byte[] tmp = new byte[4];

		ByteArrayInputStream input = new ByteArrayInputStream(inSrc);
		ByteArrayOutputStream output = new ByteArrayOutputStream();

		try {
			while (input.available() > 0) {
				int len = input.read(tmp); // 읽어 온 데이터 개수를 반환한다.
				output.write(tmp, 0, len); // 읽어 온 만큼만 write한다.
			}
		} catch (IOException e) {}
		
		outSrc = output.toByteArray();
		
		System.out.println(Arrays.toString(inSrc));
		System.out.println(Arrays.toString(tmp));
		System.out.println(Arrays.toString(outSrc));
	}
}
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[8, 9, 6, 7]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

이전 예제의 문제점을 수정한 예제이다.    
출력할 때, tmp에 저장된 모든 내용을 출력하는 대신 값을 읽어온 만큼만 출력하도록 변경하였다.     

## FileInputStream과 FileOutputStream

FileInputStream/FileOutputStream은 파일 입출력을 하기 위한 스트림이다.    
실제 프로그래밍에서 많이 사용되는 스트림 중의 하나이다.

```java
FileInputStream과 FileOutputStream의 생성자

FileInputStream(String name)
지정된 파일 이름name을 가진 실제 파일과 연결된 FileInputStream을 생성한다.

FileInputStream(File file)
파일의 이름이 String이 아닌 File인스턴스로 지정해주어야 하는 점을 제외하고
FileInputStream(String name)과 같다.

FileInputStream(FileDescriptor fdObj)
파일 디스크립터fdObj로 FileInputStream을 생성한다.

FileOutputStream(String name)
지정된 파일이름(name)을 가진 실제 파일과의 연결된 FileOutputStream을 생성한다.

FileOutputStream(String name, boolean append)
지정된 파일이름을 가진 실제 파일과 연결된 FileOutputStream을 생성한다.
두번째 인자인 append를 true로 하면, 출력시 기존의 파일내용의 마지막에 덧붙인다.
false면 기존의 파일내용을 덮어쓰게 된다.

FileOutputStream(File file)
파일의 이름을 String이 아닌 File인스턴스로 지정해주어야 하는 점을 제외하고
FileOutputStream(String name)과 같다.

FileOutputStream(File file, boolean append)
파일의 이름을 String이 아닌 File인스턴스로 지정해주어야 하는 점을 제외하고
FileOutputStream(String name, boolean append)와 같다.

FileOutputStream(FileDescriptor fdObj)
파일 디스크립터fdObj로 FileOutputStream을 생성한다.
```

```java
import java.io.FileInputStream;
import java.io.IOException;

public class FileViewer {

	public static void main(String[] args) throws IOException {
		FileInputStream fis = new FileInputStream("123.txt");
		int data = 0;
		
		while((data=fis.read())!=-1) {
			char c = (char)data;
			System.out.println(c);
		}
	}
}
```
read()의 반환값이 int형(4byte)이긴 하지만, 더 이상 입력값이 없음을 알리는 -1을 제외하고는      
0 ~ 255(1byte)범위의 정수값이기 때문에, char형(2byte)으로 변환한다 해도 손실되는 값은 없다.    
read()가 한 번에 1byte씩 파일로부터 데이터를 읽어 들이긴 하지만,       
데이터의 범위가 십진수로 0 ~ 255(16진수로는 0x00~0xff)범위의 정수값이고,      
또 읽을 수 있는 입력값이 더 이상 없음을 알릴 수 있는 값-1도 필요하다.    
그래서 다소 크긴 하지만 정수형 중에서는 연산이 가장 효율적이고 빠른 int형 값을 반환하도록 한 것이다.    

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;

public class FileCopy {

	public static void main(String[] args) {
		try {
			FileInputStream fis = new FileInputStream("123.txt");
			FileOutputStream fos = new FileOutputStream("456.txt");
			
			int data = 0;
			while((data = fis.read())!=-1) {
				fos.write(data);
			}
			fis.close();
			fos.close();
		} catch (Exception e) {}
	}
}
```

단순히 123.txt의 내용을 read()로 읽어서, write(int b)로 456.txt에 출력한다.    
이처럼 텍스트파일을 다루는 경우에는 FileInputStream/FileOutputStream보다 문자기반의 스트림인    
FileReader/FileWriter를 상ㅇ하는 것이 좋다.    

> 기존의 파일에 새로운 내용을 추가하려면, FileOutputStream fos = new FileOutputStream(456.txt, true);      
> 와 같이 생성자의 두번째 매개변수의 값을 true로 해야한다.


# 바이트기반의 보조스트림

## FilterInputStream과 FilterOutputStream

FilterInputStream/FilterOutputStream은 InputStream/OutputStream의 자소이면서 모든 보조스트림의 조상이다.    
보조스트림은 자체적으로 입출력을 수행할 수 없기 때문에 기반스트림을 필요로 한다.   
아래는 FilterInputStream/FilterOutputStream의 생성자다.    

> protected FilterInputStream(InputStream in)     
> public FilterOutputStream(OutputStream out)

FilterInputStream/FilterOutputStream의 모든 메서드는 단순히 기반스트림의 메서드를 그대로 호출할 뿐이다.    
FilterInputStream/FilterOutputStream자체로는 아무런 일도 하지 않음을 의미한다.    
FilterInputStream/FilterOutputStream은 상속을 통해 원하는 작업을 수행하도록 읽고 쓰는 메서드를 오버라이딩해야 한다.    

```java
public class FilterInputStream extends InputStream{
	protected volatile InputStream in;
	protected FilterInputStream(InputStream in) {
		this.in = in;
	}
	public int read() throws IOException{
		return in.read();
	}
	...
}
```
생성자 FilterInputStream(InputStream in)은 접근 제어자가 protected이기 때문에      
인스턴스를 생성해서 사용할 수 없고 상속을 통해서 오버라딩되어야 한다.    
FilterInputStream/FilterOutputStream을 상속받아서 기반스트림에 보조기능을 추가한 보조스트림 클래스는 다음과 같다.    

> FilterInputStream의 자손: BufferedInputStream, DataInputStream, PushbackInputStream 등    
> FilterOutputStream의 자손: BufferedOutputStream, DataOutputStream, PrintStream 등

## BufferedInputStream과 BufferedOutputStream
BufferedIntputStream/BufferedOutputStream은 스트림의 입출력 효율을 높이기 위해 버퍼를 사용하는 보조스트림이다.     
한 바이트씩 입출력하는 것 보다는 버퍼(바이트배열)를 이용해서     
한 번에 여러 바이트를 입출력하는 것이 빠르기 때문에 대부분의 입출력 작업에 사용된다.      

```
BufferedInputStream의 생성자

생성자: BufferedInputStream(InputStream in, int size)
설명: 주어진 InputStream인스턴스를 입력소스(input source)로 하며 지정된 크기(byte단위)의 버퍼를 갖는 BufferedInputStream인스턴스를 생성한다.

생성자: BufferedInputStream(InputStream in)
설명: 주어진 InputStream의 인스턴스를 입력소스로 하며 버퍼의 크기를 지정해주지 않으므로 기본적으로 8192byte 크기의 버퍼를 갖게 된다
```

BufferedInputStream의 버퍼크기는 입력소스로부터 한 번에 가져올 수 있는 데이터의 크기로 지정하면 좋다.     
보통 입력소스가 파일인 경우 8192(8k)정도의 크기로 하는 것이 보통이며, 버퍼의 크기를 변경해가면서 테스트하면 최적의 버퍼크기를 알아낼 수 있다.    
프로그램에서 입력소스로부터 데이터를 읽기 위해 처음으로 read메서드를 호출하면,     
BufferedInputStream은 입력소스로 부터 버퍼 크기만큼의 데이터를 읽어다 자신의 내부 버퍼에 저장한다.     
이제 프로그램에서는 BufferedInputStream의 버퍼에 저장된 데이터를 읽으면 되는 것이다.    
외부의 입력소스로 부터 읽는 것보다 내부의 버퍼로 부터 읽는 것이 훨씬 빠르기 때문에 그만큼 작업 효율이 높아진다.    
프로그램에서 버퍼에 저장된 모든 데이터를 다 읽고 그 다음 데이터를 읽기위해 read메서드가 호출되면,     
BufferedInputStream은 입력소스로부터 다시 버퍼크기 만큼의 데이터를 읽어다 버퍼에 저장해 놓는다. 이 작업이 계속해서 반복된다.    

```
BufferedOutputStream의 생성자와 메서드

생성자: BufferedOutputStream(OutputStream out, int size)
설명: 주어진 OutputStream인스턴스를 출력소스(Output Source)로하며 지정된 크기(단위 byte)의 버퍼를 갖는 BufferedOutputStream인스턴스를 생성한다.

생성자: BufferedOutputStream(OutputStream out)
설명: 주어진 OutputStream인스턴스를 출력소스로 하며 버퍼의 크기를 지정해주지 않으므로 기본적으로 8192byte 크기의 버퍼를 갖게 된다.

메서드: flush()
설명: 버퍼의 모든 내용을 출력소스에 출력한 다음, 버퍼를 비운다.

메서드: close()
설명: flush()를 호출해서 버퍼의 모든 내용을 출력소스에 출력하고, BufferedOutputStream인스턴스가 사용하던 모든 자원을 반환한다.
```

BufferedOutputStream 역시 버퍼를 이용해서 출력소스와 작업을 하게 되는데,    
입력소스로부터 데이터를 읽을 때와는 반대로 프로그램에서 write메서드를 이용한 출력이 BufferedOutputStream의 버퍼에 저장된다.     
버퍼가 가득 차면, 그때 버퍼의 모든 내용을 출력소스에 출력한다.    
그리고는 버퍼를 비우고 다시 프로그램으로부터의 출력을 저장할 준비를 한다.   
버퍼가 가득 찼을 때만 출력소스에 출력을 하기 때문에, 마지막 출력부분이 출력소스에 쓰이지 못하고        
BufferedOutputStream의 버퍼에 남아있는 채로 프로그램이 종료될 수 있다는 점을 주의해야한다.     
그래서 프로그램에서 모든 출력작업을 마친 후 BufferedOutputStream에 close()나 flusch()를 호출해서     
마지막에 버퍼에 있는 모든 내용이 출력소스에 출력되도록 해야 한다.     

> BufferedOutputStream의 close()는 flush()를 호출하여 버퍼의 내용을 출력 스트림에 쓰도록 한 후,     
> BufferedOutputStream인스턴스의 참조변수에 null을 지정함으로써 사용하던 자원들이 반환되게 한다. 

```java
import java.io.*;

public class BufferedOutputStreamEx1 {

	public static void main(String[] args) {
		try {
			FileOutputStream fos = new FileOutputStream("123.txt");
			//BufferedOutputStream의 버퍼 크기를 5로 한다.
			BufferedOutputStream bos = new BufferedOutputStream(fos, 5);
			//파일 123.txt에 1부터 9까지 출력한다.
			for(int i = '1'; i<='9'; i++)
				bos.write(i);		
			
			fos.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
123.txt: 12345
```

크기가 5인 BufferedOutputStream을 이용해서 파일 123.txt에 1부터 9까지 출력하는 예제인데    
결과를 보면 5까지만 출력된 것을 알 수 있다.     
그 이유는 버퍼에 남아있는 데이터가 출력되지 못한 상태로 프로그램이 종료되었기 때문이다.     
이 예제에서 fos.close()를 호출해서 스트림을 닫아주기는 했지만,     
이렇게 해서는 BufferedOutputStream의 버퍼에 있는 내용이 출력되지 않는다.    
bos.close(); 와 같이 해서 BufferedOutputStream의 close()를 호출해 주어야 버퍼에 남아있던 모든 내용이 출력된다.     
BufferedOutputStream의 close()는 기반 스트림인 FileOutputStream의 close()를 호출하기 때문에     
FileOutputStream의 close()는 따로 호출해주지 않아도 된다.    


아래의 코드는 BufferedOutputStream의 조상인 FilterOutputStream의 소스코드인데    
FilterOutputStream에 정의된 close()는 flush()를 호출한 다음에 기반스트림의 close()를 호출하는 것을 알 수 있다.     
BufferedOutputStream은 FilterOutputStream의 close()를 오버라이딩없이 그대로 상속받는다.   

```java
public class FilterOutputStream extends OutputStream {
	protected OutputStream out;
	public FilterOutputStream(OutputStream out) {
		this.out = out;
	}
	...
	public void close() throws IOException {
		try{
			flush();
		} catch(IOException ignored) {}
		out.close();	//기반스트림의 close()를 호출한다.
	}
}
```
이처럼 보조스트림을 사용한 경우에는 기반스트림의 close()나 flush()를 호출할 필요없이     
단순히 보조스트림의 close()를 호출하기만 하면 된다.

## DataInputStream과 DataOutputStream

DataInputStream/DataOutputStream도 각각 FilterInputStream/FilterOutputStream의 자손이며    
DataInputStream은 DataInput인터페이스를, DataOutputStream은 DataOutput인터페이스를 각각 구현하였기 때문에,    
데이터를 읽고 쓰는데 있어서 byte단위가 아닌, 8가지 기본 자료형의 단위로 읽고 쓸 수 있다는 장점이 있다.    
DataOutputStream이 출력하는 형식은 각 기본 자료형 값을 16진수로 표현하여 저장한다.   
예를 들어 int값을 출력한다면, 4byte의 16진수로 출력된다.    
각 자료형의 크기가 다르므로, 출력한 데이터를 다시 읽어 올 때는 출력했을 때의 순서를 염두에 두어야 한다.

```
DataInputStream의 생성자와 메서드

DataInputStream(InputStream in)
주어진 InputStream인스턴스를 기반스트림으로 하는 DataInputStream인스턴스를 생성한다.

boolean readBoolean()
byte readByte()
char readChar()
short readShort()
int readInt()
long readLong()
float readFloat()
double readDouble()
int readUnsignedByte()
int readUnsignedShort()
각 타입에 맞게 값을 읽어온다.
더 이상 읽을 값이 없으면 EOFException을 발생시킨다.

void realFully(byte[] b)
void readFully(byte[] b, int off, int len)
입력스트림에서 지정된 배열의 크기만큼 또는 지정된 위치에서 len만큼 데이터를 읽어온다.
파일의 끝에 도달하면 EOFException이 발생하고, I/O에러가 발생하면 IOException n이 발생한다.

String readUTF()
UTF-8형식으로 쓰여진 문자를 읽는다.
더 이상 읽을 값이 없으면 EOFException이 발생한다.

static String readUTF(DataInput n)
입력스트림in에서 UTF-8형식의 유니코드를 읽어온다.

int skipByte(int n)
현재 읽고 있는 위치에서 지정된 숫자n 만큼을 건너뛴다.
```

```
DataOutputStream의 생성자와 메서드

DataOutputStream(OutputStream out)
주어진 OutputStream인스턴스를 기반스트림으로 하는 DataOutputStream인스턴스를 생성한다.

void writeBoolean(boolean b)
void writeByte(int b)
void writeChar(int c)
void writeChars(String s)
void writeShort(int s)
void writeInt(int i)
void writeLong(long l)
void writeFloat(float f)
void writeDouble(double b)
각 자료형에 알맞은 값들을 출력한다.

void writeUTF(String s)
UTF형식으로 문자를 출력한다.

void writeChars(String s)
주어진 문자열을 출력한다. writeChar(int c)메서드를 여러번 호출한 결과와 같다.

int size()
지금까지 DataOutputStream에 쓰여진 byte의 수를 알려준다.
```

```java
public class DataOutputStreamEx1 {

	public static void main(String[] args) {
		FileOutputStream fos = null;
		DataOutputStream dos = null;
		
		try {
			fos = new FileOutputStream("sample.dat");
			dos = new DataOutputStream(fos);
			dos.writeInt(10);
			dos.writeFloat(20.0f);
			dos.writeBoolean(true);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
```
FileOutputStream을 기반으로 하는 DataOutputStream을 생성한 후,       
DataOutputStreaam의 메서드들을 이용해서 sample.dat파일에 값들을 출력했다.   
이 때 출력한 값들은 이진 데이터로 저장된다.      
문자 데이터가 아니므로 문서 편집기로 sample.dat를 열어 봐도 알 수 없는 글자들로 이루어져 있을 것이다.     
파일을 16진코드로 볼 수 있는 UltraEdit과 같은 프로그램이나 ByteArrayOutputStream을 사용하면 이진데이터를 확인할 수 있다.    

```java
public class DataOutputStreamEx2 {

	public static void main(String[] args) {
		ByteArrayOutputStream bos = null;
		DataOutputStream dos = null;
		
		byte[] result = null;
		
		try {
			bos = new ByteArrayOutputStream();
			dos = new DataOutputStream(bos);
			dos.writeInt(10);
			dos.writeFloat(20.0f);
			dos.writeBoolean(true);
			
			result = bos.toByteArray();
			String[] hex = new String[result.length];
			
			for(int i=0; i<result.length;i++) {
				if(result[i]<0) {
					hex[i] = String.format("%02x", result[i]+256);
				}else {
					hex[i] = String.format("%02x", result[i]);
				}
			}
			System.out.println(Arrays.toString(result));
			System.out.println(Arrays.toString(hex));
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}

[0, 0, 0, 10, 65, -96, 0, 0, 1]
[00, 00, 00, 0a, 41, a0, 00, 00, 01]
```

이전의 예제를 변경해서 FileOutputStream대신 ByteArrayOutputStream을 사용하였다.    
결과를 보며 첫 번째 4byte인 0,0,0,10은 writeInt(10)에 의해서 출력된 값이고,     
두 번재 4byte인 65,-96,0,0은 writeFloat(20.0f)에 의해서 출력된 것이다.   
그리고 마지막 1 byte인 1은 writeBoolean(true)에 의해서 출력된 것이다.    


모든 bit의 값이 1인 1byte인 데이터가 있다고 할 때, 왼쪽에서 첫 번째 비트를 부호로 인식하지 않으면    
부호 없는 1byte가 되어 범위는 0 ~ 255이므로 이 값은 최대값인 255가 되지만,     
부호로 인식하는 경우 범위는 -128 ~ 127이 되고, 이 값은 0보다 1작은 값인 -1이 된다.    
결국 같은 데이터이지만 자바의 자료형인 byte의 범위가 부호 있는 1byte 정수의 범위인 -128 ~ 127이기 때문에     
-1로 인식한다는 것이다. 그래서 이 값을 0 ~ 255사이의 값으로 변환하려면 256을 더해줘야 한다.     
예를 들어 -1의 경우 -1 + 255 = 255가 된다. 그리고 반대의 경우 256을 빼면 된다.    
그 다음에 String.format()을 사용해서 10진 정수를 16진 정수로 변환하여 출력했다.    
이처럼 ByteArrayInputStream/ByteArrayOutputStream을 사용하면 byte단위의 데이터 변환 및 조작이 가능하다.    

> InputStream의 read()는 반환타입이 int이며 0 ~ 255의 값을 반환하므로 256을 더하거나 뺄 필요가 없다.     
> 반면에 read(byte[] b)와 같이 byte배열을 사용하는 경우 상황에 따라 0 ~ 255범위의 값으로 변환해야할 필요가 있다.

사실 DataInputStream에 의해서 어떻게 저장되는지 몰라도 DataOutputStream의 write메서드들로    
기록한 데이터는 DataInputStream의 read메서드들로 읽기만 하면 된다.    
이 때 여러 가지 종류의 자료형으로 출력을 했다면 읽을 때는 반드시 쓰인 순서대로 읽어야 한다.

```java
public class DataInputStreamEx1 {
	public static void main(String[] args) {
		try {
			FileInputStream fis = new FileInputStream("sample.dat");
			DataInputStream dis = new DataInputStream(fis);
			
			System.out.println(dis.readInt());
			System.out.println(dis.readFloat());
			System.out.println(dis.readBoolean());
			dis.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
10
20.0
true
```

sample.dat파일로부터 데이터를 읽어 올 때, 아무런 변환이나 자릿수를 셀 필요없이 단순히     
readInt()와 같이 읽어 올 데이터의 타입에 맞는 메서드를 사용하기만 하면 된다.    


문자로 데이터를 저장하면, 다시 데이터를 읽어 올 때 문자들을 실제 값으로 변환하는,    
예를 들면 문자열 "100"을 숫자 100으로 변환하는, 과정을 거쳐야 하고,   
또 읽어야 할 데이터의 개수를 결정해야하는 번거로움이 있다.     
하지만 이처럼 DataInputStream과 DataOutputStream을 사용하면, 데이터를 변환할 필요도 없고,    
자리수를 세어서 따지지 않아도 되므로 편리하고 빠르게 데이터를 저장하고 읽을 수 있게 된다.    

```java
public class DataOutputStreamEx3 {

	public static void main(String[] args) {
		int[] score = {100,90,95,85,50};
		
		try {
			FileOutputStream fos = new FileOutputStream("score.dat");
			DataOutputStream dos = new DataOutputStream(fos);
			
			for(int i =0; i<score.length;i++)
				dos.writeInt(score[i]);
			
			dos.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
int형 배열 score의 값들을 DataOutputStream을 이용해서 score.dat파일에 출력하는 예제이다.    
type명령으로 score.dat의 내용을 보면 숫자가 아니라 문자들이 나타나는데,    
그 이유는 type명령이 파일의 내용을 문자로 변환해서 보여주기 때문이다.      
int의 크기가 4 byte이므로 모두 20byte의 데이터가 저장되어 있다.

```java
import java.io.*;

public class DataInputStreamEx2 {
	public static void main(String[] args) {
		int sum = 0;
		int score = 0;
		
		FileInputStream fis = null;
		DataInputStream dis = null;
		
		try {
			fis = new FileInputStream("score.dat");
			dis = new DataInputStream(fis);
			
			while(true) {
				score = dis.readInt();
				System.out.println(score);
				sum += score;
			}
		} catch (EOFException e) {
			System.out.printf("점수의 총합은 %d입니다.", sum);
		} catch (IOException ie) {
			ie.printStackTrace();
		} finally {
			try{
				if(dis!=null)
					dis.close();				
			} catch (IOException ie) {
				ie.printStackTrace();
			}
		}
	}
}
100
90
95
85
50
점수의 총합은 420입니다.
```

DataInputStream의 readInt()와 같이 데이터를 읽는 메서드는 더 이상 읽을 데이터가 없으면 EOFException을 발생시킨다.    
그래서 다른 입력스트림들과는 달리 무한반복문과 EOFException을 처리하는 catch문을 이용해서 데이터를 읽는다.    
원래 while문으로 작업을 마친 후에 스트림을 닫아 줘야 하는데,       
while문이 무한 반복문이기 때문에 finally블럭에서 스트림을 닫도록 처리하였다.     
참조변수 dis가 null일 때 close()를 호출하면 NullPointerException이 발생하므로    
if문을 사용해서 dis가 null인지 체크한 후에 close()를 호출해야 한다.   
그리고 close()는 IOException을 발생시킬 수 있으므로 try-catch블럭으로 감싸주었다.    


지금까지는 try블럭내에서 스트림을 닫아주었지만, 작업도중에 예외가 발생해서 스트림을 닫지 못하고     
try블럭을 빠져나갈 수 있기 때문에 이처럼 finally블럭을 이용해서 스트림을 닫아주는 것이 더 확실한 방법이다.    
사실 프로그램이 종료될 때, 가비지 컬렉터가 사용하던 자원들을 모두 해제 해주기 때문에       
이렇게 간단한 예제에서는 스트림을 닫지 않아도 별문제가 되지는 않는다.     
그래도 가능하면 스트림을 사용한 직후에 바로 닫아서 자원을 반환하는 것이 좋다.     
JDK1.7부터는 try-with-resources문을 이용해서 close()를 직접 호출하지 않아도 자동호출되도록 할 수 있다.    
아래 예제는 앞의 예제를 try-with-resources문을 이용해서 변경한 것이다.

```java
public class DataInputStreamEx3 {
	public static void main(String[] args) {
		int sum = 0;
		int score = 0;
		
		try(FileInputStream fis = new FileInputStream("score.dat");
			DataInputStream dis = new DataInputStream(fis)) {
			
			while(true) {
				score = dis.readInt();
				System.out.println(score);
				sum += score;
			}
		} catch (EOFException e) {
			System.out.printf("점수의 총합은 %d입니다.", sum);
		} catch (IOException ie) {
			ie.printStackTrace();
		}
	}
}
```

## SequenceInputStream
SequenceInputStream은 여러 개의 입력스트림을 연속적으로 연결해서 하나의 스트림으로부터     
데이터를 읽는 것과 같이 처리할 수 있도록 도와준다.     
SequenceInputStream의 생성자를 제외하고 나머지 작업은 다른 입력스트림과 다르지 않다.    
큰 파일을 여러 개의 작은 파일로 나누었다가 하나의 파일로 합치는 것과 같은 작업을 수행할 때 사용하면 좋다.    
> SequenceInputStream은 다른 보조스트림들과는 달리 FilterInputStream의 자손이 아닌     
> InputStream을 바로 상속받아서 구현하였다.

```
SequenceInputStream(Enumeration e)
Enumeration에 저장된 순서대로 입력스트림을 하나의 스트림으로 연결한다.
SequenceInputStream(InputStream s1, InputStream s2)
두 개의 입력스트림을 하나로 연결한다.
```

Vector에 연결할 입력스트림들을 저장한 다음 Vector의 Enumeration elements()를 호출해서 생성자의 매개변수로 사용한다.

```java
사용 예1
Vector files = new Vector();
files.add(new FileInputStream("file.001"));
files.add(new FileInputStream("file.002"));
SequenceInputStream in = new SequenceInputStream(files.elements());

사용 예2
FileInputStream file1 = new FileInputStream("file.001");
FileInputStream file2 = new FileInputStream("file.002");
SequenceInputStream in = new SequenceInputStream(file1, file2);
```

```java
public class SequenceInputStreamEx {

	public static void main(String[] args) {
		byte[] arr1 = {0,1,2};
		byte[] arr2 = {3,4,5};
		byte[] arr3 = {6,7,8};
		byte[] outSrc = null;
		
		Vector v = new Vector();
		v.add(new ByteArrayInputStream(arr1));
		v.add(new ByteArrayInputStream(arr2));
		v.add(new ByteArrayInputStream(arr3));
		
		SequenceInputStream input = new SequenceInputStream(v.elements());
		ByteArrayOutputStream output = new ByteArrayOutputStream();
		
		int data = 0;
		
		try {
			while((data = input.read())!=-1) {
				output.write(data);
			}
		} catch (IOException e) {}
		outSrc = output.toByteArray();
		
		System.out.println(Arrays.toString(arr1));
		System.out.println(Arrays.toString(arr2));
		System.out.println(Arrays.toString(arr3));
		System.out.println(Arrays.toString(outSrc));
	}
}
[0, 1, 2]
[3, 4, 5]
[6, 7, 8]
[0, 1, 2, 3, 4, 5, 6, 7, 8]
```

3개의 ByteArrayInputStream을 Vector와 SequenceInputStream을 이용해서 하나의 입력스트림처럼 다룰 수 있다.    
Vector에 저장된 순서대로 입력되므로 순서에 주의하여야 한다.

## PrintStream

PrintStream은 데이터를 기반스트림에 다양한 형태로 출력할 수 있는       
print, printf, println와 같은 메서드를 오버로딩하여 제공한다.     
PrintStream은 데이터를 적절한 문자로 출력하는 것이기 때문에 문자기반 스트림의 역할을 수행한다.    
그래서 JDK1.1에서 부터 PrintStream보다 향상된 기능의 문자기반 스트림인 PrintWriter가 추가되었으나    
그 동안 빈번히 사용되던 System.out이 PrintStream이다 보니 둘 다 사용할 수 밖에 없게 되었다.    
PrintStream과 PrintWriter는 거의 같은 기능을 가지고 있지만 PrintWriter가 PrintStream에 비해    
다양한 언어의 문자를 처리하는데 적합하기 때문에 가능하면 PrintWriter를 사용하는 것이 좋다.    

```
PrintStream의 생성자와 메서드

생성자
PrintStream(File file)
PrintStream(File file, String csn)
PrintStream(OutputStream out)
PrintStream(OutputStream out, boolean autoFlush)
PrintStream(OutputStream out, boolean autoFlush, String encoding)
PrintStream(String fileName)
PrintStream(String fileName, String csn)
지정된 출력스트림을 기반으로 하는 PrintStream인스턴스를 생성한다.
autoFlush의 값을 true로 하면 println메서드가 호출되거나
개행문자가 출력될 때 자동으로 flush된다. 기본값은 false이다.

메서드
boolean checkError()
스트림을 flush하고 에러가 발생했는지를 알려준다.

void print(...)
void println(...)
인자로 주어진 값을 출력소스에 문자로 출력한다.
println메서드는 출력 후 줄바꿈을 하고, print메서드는 줄을 바꾸지 않는다.

PrintStream printf(String format, Object...args)
정형화된formatted출력을 가능하게 한다.

protected void setError()
작업 중에 오류가 발생했음을 알린다.
setError()를 호출한 후에 checkError()를 호출하면 true를 반환한다.
```

print()나 println()을 이용해서 출력하는 중에 PrintStream의 기반스트림에서 IOException이 발생하면    
checkError()를 통해서 인지할 수 있다.     
println()이나 print()는 예외를 던지지 않고 내부에서 처리하도록 정의하였는데,   
그 이유는 println()과 같은 메서드가 자주 사용되는 것이기 떄문이다.    
만일 println()이 예외를 던지도록 정의되었다면 println()을 사용하는 모든 곳에 try-catch문을 사용해야 할 것이다.     

```java
public class PrintStream extends FilterOutputStream implements Appendable, Closeable{
	...
	private boolean trouble = false;
	
	public void print(int i) {
		write(String.valueOf(i));	//write(i+"");와 같다.
	}
	private void write(String s){
		try{
			...
		} catch(IOException x) {
			trouble = true;
		}
	}
	...
	public boolean checkError(){
		if(out != null) flush();
		return trouble;
	}
}
```
> i+""와 String.valueOf(i)는 같은 결과를 얻지만 String.valueOf(i)가 성능이 더 좋다.

# 문자기반 스트림
문자데이터를 다루는데 사용된다는 것을 제외하면 바이트기반 스트림과 사용방법은 거의 같다.

## Reader와 Writer
문자기반 스트림의 조상은 Reader/Writer이다.    
byte배열 대신 char배열을 사용한다는 것 외에는 InputStream/OutputStream의 메서드와 다르지 않다.

```
Reader의 메서드

abstract void close()
입력스트림을 닫음으로써 사용하고 있던 자원을 반환한다ㅏ.

void mark(int readlimit)
현재 위치를 표시해놓는다. 후에 reset()에 의해서 표시해놓은 위치로 다시 돌아갈 수 있다.

boolean markSupported()
mark()와 reset()을 지원하는지를 알려준다.

int read()
입력소스로부터 하나의 문자를 읽어온다.
char의 범위인 0~65535범위의 정수를 반환하며, 입력스트림의 마지막 데이터에 도달하면 -1을 반환한다.

int read(char[] c)
입력소스로부터 매개변수로 주어진 배열 c의 크기만큼 읽어서 배열 c에 저장한다. 읽어 온 데이터의 개수 또는 -1을 반환한다.

abstract int read(char[] c, int off, int len)
입력소스로부터 최대 len개의 문자를 읽어서, 배열 c의 지정된 위치off부터 읽은 만큼 저장한다.
읽어 온 데이터의 개수 또는 -1을 반환한다.

int read(CharBuffer target)
입력소스로부터 읽어서 문자버퍼target에 저장한다.

boolean ready()
입력소스로부터 데이터를 읽을 준비가 되었는지 알려 준다.

void reset()
입력소스에서의 위치를 마지막으로 mark()가 호출되었던 위치로 되돌린다.

long skip(long n)
현재 위치에서 주어진 문자 수n만큼 건너뛴다.
```

```
Writer의 메서드

Writer append(char c)
지정된 문자를 출력소스에 출력한다.

Writer append(CharSequence c)
지정된 문자열CharSequence을 출력소스에 출력한다.

Writer append(CharSequence c, int start, int end)
지정된 문자열CharSequence의 일부를 출력소스에 출력(CharBuffer, String, StringBuffer가 CharSequence를 구현)

abstract void close()
출력스트림을 닫음으로써 사용하고 있던 자원을 반환한다.

abstract void flush()
스트림의 버퍼에 있는 모든 내용을 출력소스에 쓴다.(버퍼가 있는 스트림에만 해당됨)

void write(int b)
주어진 값을 출력소스에 쓴다.

void write(char[] c)
주어진 배열 c에 저장된 모든 내용을 출력소스에 쓴다.

abstract void write(char[] c, int off, int len)
주어진 배열 c에 저장된 내용 중에서 off번째부터 len길이만큼만 출력소스에 쓴다.

void write(String str)
주어진 문자열str을 출력소스에 쓴다.

void write(String str, int off, int len)
주어진 문자열str의 일부를 출력소스에 쓴다.(off번째 문자부터 len개 만큼의 문자열)
```

문자기반 스트림은 단순히 2 byte로 스트림을 처리하는 것만을 의미하지는 않는다.    
문자 데이터를 다루는데 필요한 또 하나의 정보는 인코딩이다.     
문자기반 스트림, 즉 Reader/Writer 그리고 그 자손들은 여러 종류의 인코딩과 자바에서 사용하는 유니코드(UTF-16)간의 변환을 자동으로 처리해준다.    
Reader는 특정 인코딩을 읽어서 유니코드로 변환하고 Writer는 유니코드를 특정 인코딩으로 변환하여 저장한다.

## FileReader와 FileWriter
FileReader/FileWriter는 파일로부터 텍스트데이터를 읽고, 파일에 쓰는데 사용된다.    

```java
public class FileReaderEx1 {

	public static void main(String[] args) {
		try {
			String fileName = "test.txt";
			FileInputStream fis = new FileInputStream(fileName);
			FileReader fr = new FileReader(fileName);
			
			int data = 0;
			//FileInputStream을 이용해서 파일내용을 읽어 화면에 출력한다.
			while((data=fis.read())!=-1) {
				System.out.println((char)data);
			}
			System.out.println();
			fis.close();
			
			//FileReader를 이용해서 파일내용을 읽어 화면에 출력한다.
			while((data=fr.read())!=-1){
				System.out.println((char)data);
			}
			System.out.println();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
```

결과에서 알 수 있듯이 FileInputStream을 사용했을 때는 한글이 꺠져서 출력된다.

```java
public class FileConversion {

	public static void main(String[] args) {
		try {
			FileReader fr = new FileReader(args[0]);
			FileWriter fw = new FileWriter(args[1]);
			
			int data = 0;
			while((data=fr.read())!=-1) {
				if(data!='\t' && data!='\n' && data!=' ' && data !='\r')
					fw.write(data);
			}
			fr.close();
			fw.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
```
파일의 공백을 모두 없애는 예제이다.    
입력스트림으로부터 읽은 데이터를 변환해서 출력스트림에 쓰는 작업의 예를 보여주기 위한 것이다.

## PipedReader와 PipedWriter
PipedReader/PipedWriter는 쓰레드 간에 데이터를 주고받을 때 사용된다.    
다른 스트릠과는 달리 입력과 출력스트림을 하나의 스트림으로 연결해서 데이터를 주고받는다는 특징이 있다.     
스트림을 생성한 다음에 어느 한쪽 쓰레드에서 connect()를 호출해서 입력스트림과 출력스트림을 연결한다.     
입출력을 마친 후에는 어느 한쪽 스트림만 닫아도 나머지 스트림을 자동으로 닫힌다.

```java
public class PipedReaderWriter {

	public static void main(String[] args) {
		InputThread inThread = new InputThread("InputThread");
		OutputThread outThread = new OutputThread("OutputThread");
		
		inThread.connect(outThread.getOutput());
		
		
		inThread.start();
		outThread.start();
	}
}
class InputThread extends Thread{
	PipedReader input = new PipedReader();
	StringWriter sw = new StringWriter();
	
	InputThread(String name){
		super(name);	//Thread(String name)
	}
	
	public void run() {
		try {
			int data=0;
			while((data=input.read())!=-1) {
				sw.write(data);
			}
			System.out.println(getName()+sw.toString());
		} catch (Exception e) {}
	}
	public PipedReader getInput() {
		return input;
	}
	public void connect(PipedWriter output) {
		try {
			input.connect(output);
		} catch (Exception e) {}
	}
}
class OutputThread extends Thread{
	PipedWriter output = new PipedWriter();
	
	OutputThread(String name){
		super(name);
	}
	
	public void run() {
		try {
			String msg = "hello";
			System.out.println(getName()+msg);
			output.write(msg);
			output.close();
		} catch (Exception e) {}
	}
	
	public PipedWriter getOutput() {
		return output;
	}
	
	public void connect(PipedReader input) {
		try {
			output.connect(input);
		} catch (Exception e) {}
	}
}
```
두 쓰레드가 PipedReader/PipedWriter를 이용해서 서로 메시지를 주고받는 예제이다.    
쓰레드를 시작하기 전에 PipedReader와 PipedWriter를 연결해야한다는 것에 유의해야 한다.    
StringWriter는 CharArrayWriter처럼 메모리를 사용하는 스트림인데     
내부적으로 String Buffer를 가지고 있어서 출력하는 내용이 여기에 저장된다.

## StringReader와 StringWriter
StringReader/StringWriter는 CharArrayReader/CharArrayWriter와 같이 입출력 대상이 메모리인 스트림이다.    
StringWriter에 출력되는 데이터는 내부의 StringBuffer에 저장되며    
StringWriter의 다음과 같은 메서드를 이용해서 저장된 데이터를 얻을 수 있다.

```
StringBuffer getBuffer()	StringWriter에 출력한 데이터가 저장된 StringBuffer를 반환한다.
String toString()	StringWriter에 출력된(StringBuffer에 저장된) 문자열을 반환한다.
```

근본적으로 String도 char배열이지만, 아무래도 char배열보다는 String으로 처리하는 것이 편리한 경우가 많다.    

```java
public class StringReaderWriterEx {

	public static void main(String[] args) {
		String inputData = "ABCD";
		StringReader input = new StringReader(inputData);
		StringWriter output = new StringWriter();
		
		int data = 0;
		
		try {
			while((data=input.read())!=-1) {
				output.write(data);
			}
		} catch (IOException e) {}
		
		System.out.println(inputData);
		System.out.println(output.toString());
		System.out.println(output.getBuffer().toString());
	}
}
ABCD
ABCD
ABCD
```

# 문자기반의 보조스트림

## BufferedReader와 BufferedWriter
BufferedReader/BufferedWriter는 버퍼를 이용해서 입출력의 효율을 높일 수 있도록 해주는 역할을 한다.   
버퍼를 이용하면 입출력의 효율이 좋아지기 때문에 사용하는 것이 좋다.    
BufferedReader의 readLine()을 사용하면 데이터를 라인단위로 읽을 수 있고
BufferedWriter는 newLine()이라는 줄바꿈 메서드를 가지고 있다.   

```java
public class BufferedReaderEx1 {

	public static void main(String[] args) {
		try {
			FileReader fr = new FileReader("BufferedReaderEx1.java");
			BufferedReader br = new BufferedReader(fr);
			
			String line = "";
			for(int i=1; (line=br.readLine())!=null; i++) {
				//";"를 포함한 라인을 출력한다.
				if(line.indexOf(";")!=-1)
					System.out.println(i+":"+line);
			}
			br.close();
		} catch (IOException e) {}
	}
}
```
BufferedReader의 readline()을 이용해서 파일의 라인 단위로 읽은 다음 indexOf()를 이용해서    
";"를 포함하고 있는지 확인하여 출력하는 예제이다.    
파일에서 특정 문자 또는 문자열을 포함한 라인을 쉽게 찾아낼 수 있음을 보여 준다.

## InputStreamReader와 OutputStreamWriter

InputStreamReader/OutputStreamWriter는 이름과 같이 바이트기반 스트림을 문자기반 스트림으로 연결시켜주는 역할을 한다.    
그리고 바이트기반 스트림의 데이터를 지정된 인코딩의 문자데이터로 변환하는 작업을 수행한다.

> InputStreamReader, OutputStreamWriter는 Reader,Writer의 자손이다.

```
InputStreamReader의 생성자와 메서드

InputStreamReader(InputStream in)
OS에서 사용하는 기본 인코딩의 문자로 변환하는 InputStreamReader를 생성한다.

InputStreamReader(InputStream in, String encoding)
지정된 인코딩을 사용하는 InputStreamReader를 생성한다.

String getEncoding()
InputStreamReader의 인코딩을 알려준다.
```

```
OutputStreamWriter의 생성자와 메서드

OutputStreamWriter(OutputStream out)
OS에서 사용하는 기본 인코딩의 문자로 변환하는 OutputStreamWriter를 생성한다.

OutputStreamWriter(OutputStream out, String encoding)
지정된 인코딩을 사용하는 OutputStreamWriter를 생성한다.

String getEncoding()
OutputStreamWriter의 인코딩을 알려준다.
```

인코딩을 지정해 주지 않는다면 OS에서 사용하는 인코딩을 사용해 파일을 해석해서 보여 주기 때문에 원래 작성된 데로 볼 수 없다.    
이와 마찬가지로 OutputStreamWriter를 이용해서 파일에 텍스트데이터를 저장할 때    
생성자 OutputStreamWriter(OutputStream out, String encoding)를 이용해서 인코딩을 지정하지 않으면    
OS에서 사용하는 인코딩으로 데이터를 저장할 것이다.    

```java
public class InputStreamReaderEx {
	public static void main(String[] args) {
		String line = "";
		
		try {
			InputStreamReader isr = new InputStreamReader(System.in);
			BufferedReader br = new BufferedReader(isr);
			
			System.out.println("사용중인 OS의 인코딩:"+isr.getEncoding());
			
			do {
				System.out.println("문장을 입력하세요. 마치려면 q");
				line = br.readLine();
				System.out.println("입력한 문장: "+line);
			} while(!line.equalsIgnoreCase("q"));
		} catch (IOException e) {}
	}
}
```

# 표준입출력과 File

## 표준입출력-System.in, System.out, System.err
표준입출력은 콘솔을 통한 데이터 입력과 콘솔로의 데이터 출력을 의미한다.    
자바에서는 표준 입출력standard IO을 위해 3가지 입출력 스트림, System.in, System.out, System.err을 제공하는데,    
이들은 자바 어플리케이션의 실행과 동시에 사용할 수 있게 자동적으로 생성되기 때문에 개발자가 별도로 스트림을 생성하지 않아도 사용이 가능하다.    


System클래스의 소스에서 알 수 있듯이 in, out, err은 System클래스에 선언된 클래스변수static변수이다.    
선언부분만 봐서는 out, in, err의 타입은 InputStream과 PrintStream이지만     
실제로는 버퍼를 이용하는 BufferedInputStream과 BufferedOutputStream의 인스턴스를 사용한다.      

```java
public final class System {
	public final static InputStream in = nullInputStream();
	public final static PrintStream out = nullPrintStream();
	public final static PrintStream err = nullPrintStream();
	...
}
```

## 표준입출력의 대상변경-setOut(), setErr(), setIn()

초기에는 System.in, System.out, System.err의 입출력대상이 콘솔화면이지만,    
setIn(), setOut(), setErr()를 사용하면 입출력을 콘솔 이외에 다른 입출력 대상으로 변경하는 것이 가능하다.    

```
메서드: static void setOut(PrintStream out)
설명: System.out의 출력을 지정된 PrintStream으로 변경

메서드: static void setErr(PrintStream err)
설명: System.err의 출력을 지정된 PrintStream으로 변경

메서드: static void setIn(InputStream in)
설명: System.in의 출력을 지정된 InputStream으로 변경
```

```java
public class StandardIOEx3 {
	public static void main(String[] args) {
		PrintStream ps = null;
		FileOutputStream fos = null;
		
		try {
			fos = new FileOutputStream("test.txt");
			ps = new PrintStream(fos);
			System.setOut(ps); 	//System.out의 출력대상을 test.txt파일로 변경
		} catch (FileNotFoundException e) {
			System.out.println("File not found");
		}
		System.out.println("Hello by System.out");
		System.err.println("Hello by System.err");
	}
}
```
System.out의 출력소스를 test.txt파일로 변경하였기 때문에 System.out을 이용한 출력은 모두 text.txt파일에 저장된다.   
그래서 실행결과에는 System.err를 이용한 출력만 나타난다.    

## RandomAccessFile
자바에서는 입력과 출력이 각각 분리되어 별도로 작업을 하도록 설계되어 있는데,    
RandomAccessFile만은 하나의 클래스로 파일에 대한 입력과 출력을 모두 할 수 있도록 되어 있다.    
RandomAccessFile은 InputStream이나 OutputStream으로부터 상속받지 않고,    
DataInput인터페이스와 DataOutput인터페이스를 모두 구현했기 때문에 읽기와 쓰기가 모두 가능하다.    
사실 DataInputStream은 DataInput인터페이스를, DataOutputStream은 DataOutput 인터페이스를 구현했다.    
이 두 클래스의 기본 자료형을 읽고 쓰기위한 메서드들은 모두 이 2개의 인터페이스에 정의되어있는 것들이다.    
따라서 RandomAccessFile클래스도 DataInputStream과 DataOutputStream처럼 기본자료형 단위로 데이터를 읽고 쓸 수 있다.     
RandomAccessFile의 가장 큰 장점은 파일의 어느 위치에나 읽기/쓰기가 가능하다는 것이다.     
다른 입출력 클래스들은 입출력소스에 순차적으로 읽기/쓰기를 하기 때문에 읽기와 쓰기가 제한적인데 반해서    
RandomAccessFile클래스는 파일에 읽고 쓰는 위치에 제한이 없다.   
이것을 가능하게 하기 위해서 내부적으로 파일 포인터를 사용하는데, 입출력 시에 작업이 수행되는 곳이 파일 포인터가 위치한 곳이 된다.    
파일 포인터의 위치는 파일의 제일 첫 부분(0)이며, 읽기 또는 쓰기를 수행할 때 마다 작업이 수행된 다음 위치로 이동하게 된다.    
순차적으로 읽기나 쓰기를 한다면, 파일 포인터를 이동시키기 위해 별도의 작업이 필요하지 않지만,    
파일의 임의의 위치에 있는 내용에 대해서 작업하고자 한다면, 먼저 파일 포인터를 원하는 위치로 옮긴 다음 작업을 해야 한다.   


현재 작업 중인 파일에서 파일 포인터의 위치를 알고 싶을 때는 getFilePointer()를 사용하면 되고,    
파일 포인터의 위치를 옮기기 위해서는 seek(long pos)나 skipByte(int n)를 사용하면 된다.

```
RandomAccessFile의 생성자와 메서드

RandomAccessFile(File file, String mode)
RandomAccessFile(String fileName, String mode)
주어진 file에 읽기 또는 읽기와 쓰기를 하기 위한 RandomAccessFile인스턴스를 생성한다.
mode의 값은 "r", "rw", "rws", "rwd"가 지정가능하다.
"r"-파일로부터 읽기만을 수행할 때
"rw"-파일에 읽기와 쓰기
"rws"와 "rwd"는 기본적으로 "rw"와 같은데, 출력내용이 파일에 지연 없이 바로 쓰이게 한다.
"rwd"는 파일 내용만, "rws"는 파일의 메타정보도 포함

FileChannel getChannel()
파일의 파일 채널을 반환한다.

FileDescriptor getFD()
파일의 파일 디스크립터를 반환

long getFilePointer()
파일 포인터의 위치를 알려 준다.

long length()
파일의 크기를 얻을 수 있다(단위 byte)

void seek(long pos)
파일 포인터의 위치를 변경한다.
위치는 파일의 첫 부분부터 pos크기만큼 떨어진 곳이다.

void SetLength(long newLength)
파일의 크기를 지정된 길이로 변경한다.

int skipBytes(int n)
지정된 수만큼의 byte를 건너뛴다.
```

> RandomAccessFile의 인스턴스를 "rw"mode로 생성할 때, 지정된 파일이 없으면 새로운 파일을 생성한다.

```java
public class RandomAccessFileEx1 {

	public static void main(String[] args) {
		try {
			RandomAccessFile raf = new RandomAccessFile("test.bat", "rw");
			System.out.println(raf.getFilePointer());
			raf.writeInt(100);
			System.out.println(raf.getFilePointer());
			raf.writeLong(100L);
			System.out.println(raf.getFilePointer());
			
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
0
4
12
```

int가 4byte이기 때문에 writeInt()를 호출한 다음 파일 포인터의 위치가 0에서 4로 바뀌었다.   
마찬가지로 8byte인 long을 출력하는 writeLong()을 출력한 후에는 파일 포인터의 위치가 4에서 12로 변경되었다.

```java
public class RandomAccessFileEx2 {
	public static void main(String[] args) {
		int[] score= { 1, 100, 90, 90,
						2, 70, 90, 100,
						3, 100, 100, 100,
						4, 70, 60, 80,
						5, 70, 90, 100
		};
		try {
			RandomAccessFile raf = new RandomAccessFile("score2.dat", "rw");
			for(int i = 0; i<score.length; i++)
				raf.writeInt(score[i]);
			
			while(true) {
				System.out.println(raf.readInt());
			}
		} catch (EOFException e) {
			//readInt()를 호출했을 때 더 이상 읽을 내용이 없으면 EOFException이 발생한다.
		} catch (IOException io) {
			io.printStackTrace();
		}
	}
}
```
이 예제는 int배열 score에 저장된 데이터를 score2.bat에 저장하고 저장된 내용을 readInt()로 읽어서 출력하는 예제이다.   
그러나 score2.dat파일은 생성되었지만 화면에는 아무것도 출력되지 않았다.    
그 이유는 writeInt()를 수행하면서 파일포인터의 위치가 파일의 마지막으로 이동되었기 때문이다.    
그 다음에 readInt()를 호출했으므로 파일의 앞부분이 아닌 마지막부분부터 읽기 시작하기 때문에 아무것도 읽지 못하고   
EOFException이 발생해서 무한반복문을 벗어나게 된다.    
그래서 아래와 같이 seek(long pos)를 이용해서 파일포인터의 위치를 다시 처음으로 이동시킨 다음 readInt()를 호출하도록 해야 한다.    

```java
raf.seek(0);
while(true){
	System.out.println(raf.readInt());
}
```

이처럼 RandomAccessFile을 rw모드로 생성해서 작업할 때는 이러한 점을 염두에 두어야 한다.

```java
public class RandomAccessFileEx3 {

	public static void main(String[] args) {
		int sum = 0;
		
		try {
			RandomAccessFile raf = new RandomAccessFile("score2.dat", "r");
			int i =4;
			
			while(true) {
				raf.seek(i);
				sum += raf.readInt();
				i+=16;
			}
		} catch (EOFException e) {
			System.out.println(sum);
		} catch (IOException io) {
			io.printStackTrace();
		}
	}
}
410
```
이전 예제에서 데이터를 저장한 score2.dat파일에서 국어과목의 점수만을 합계를 내는 예제이다.     
한 학생의 데이터가 번호와 3과목의 점수로 모두 4개의 int값(4 * 4=16byte)으로 되어 있기 때문에    
i+16과 같이 파일 포인터의 값을 16씩 증가시켜가면서 readInt()를 호출 했다.

## File
자바에서는 File클래스를 통해서 파일과 디렉토리를 다룰 수 있도록 하고 있다.   
그래서 File인스턴스는 파일일 수도 있고 디렉토리일 수도 있다.

```
File의 생성자와 경로와 관련된 메서드

File(String fileName)
주어진 문자열fileName을 이름으로 갖는 파일을 위한 File인스턴스를 생성한다.
파일 뿐만 아니라 디렉토리도 같은 방법으로 다룬다.
여기서 fileName은 주로 경로path를 포함해서 지정해주지만,
파일 이름만 사용해도 되는데 이 경우 프로그램이 실행되는 위치가 경로로 간주된다.

File(String pathName, String fileName)
File(File pathName, String fileName)
파일의 경로와 이름을 따로 분리해서 지정할 수 있도록 한 생성자.
이 중 두 번째 것은 경로를 문자열이 아닌 File인스턴스인 경우를 위해서 제공된 것이다.

File(URI uri)
지정된 uri로 파일을 생성

String getName()
파일이름을 String으로 반환

String getPath()
파일의 경로path를 String으로 반환

String getAbsolutePath()
File getAbsoluteFile()
파일의 절대경로를 String, File로 반환

String getParent()
File getParentFile()
파일의 조상 디렉토리를 String, File로 반환

String getCanonicalPath()
File getCanonicalPath()
파일의 정규경로를 String, File로 반환
```

```
경로와 관련된 File의 멤버변수

static String pathSeparator
static char pathSeparatorChar
OS에서 사용하는 경로 구분자. 윈도우";", 유닉스":"

static String separator
static cher separatorChar
OS에서 사용하는 이름 구분자. 윈도우"\" 유닉스"/"
```

파일의 경로와 디렉토리나 파일의 이름을 구분하는데 사용되는 구분자가 OS마다 다를 수 있기 때문에,    
OS독립적으로 프로그램을 작성하기 위해서는 반드시 위의 멤버변수들을 이용해야한다.    
만일 윈도우에서 사용하는 구분자를 코드에 직접 적어놓았다면 다른 OS에서는 오류를 일으킬 수 있다.

```java
public class FileEx1 {

	public static void main(String[] args) throws IOException {
		File f = new File("C:\\Users\\kojie\\Downloads\\class\\github\\practice_\\-\\practice\\src\\tmp\\FileEx1.java");
		String fileName = f.getName();
		int pos = fileName.lastIndexOf(".");
		
		System.out.println("경로를 제외한 파일이름:"+f.getName());
//경로를 제외한 파일이름:FileEx1.java
		System.out.println("확장자를 제외한 파일이름:"+fileName.substring(0, pos));
//확장자를 제외한 파일이름:FileEx1
		System.out.println("확장자-"+fileName.substring(pos+1));
//확장자-java
		
		System.out.println("경로를 포함한 파일이름:"+f.getPath());
//경로를 포함한 파일이름:C:\Users\kojie\Downloads\class\github\practice_\-\practice\src\tmp\FileEx1.java
		System.out.println("파일의 절대경로:"+f.getAbsolutePath());
//파일의 절대경로:C:\Users\kojie\Downloads\class\github\practice_\-\practice\src\tmp\FileEx1.java
		System.out.println("파일의 정규경로:"+f.getCanonicalPath());
//파일의 정규경로:C:\Users\kojie\Downloads\class\github\practice_\-\practice\src\tmp\FileEx1.java
		System.out.println("파일이 속해있는 디렉토리-"+f.getParent());
//파일이 속해있는 디렉토리-C:\Users\kojie\Downloads\class\github\practice_\-\practice\src\tmp
		System.out.println();
		System.out.println("File.pathSeparator-"+File.pathSeparator);
//File.pathSeparator-;
		System.out.println("File.pathSeparator.Char-"+File.pathSeparatorChar);
//File.pathSeparator.Char-;
		System.out.println("File.separator-"+File.separator);
//File.separator-\
		System.out.println("File.separator-"+File.separatorChar);
//File.separator-\
		System.out.println();
		System.out.println("user.dir="+System.getProperty("user.dir"));
//user.dir=C:\Users\kojie\Downloads\class\github\practice_\-\practice
		System.out.println("sun.boot.class.path="+System.getProperty("sun.boot.class.path"));
//sun.boot.class.path=null
	}
}
```
File인스턴스를 생성하고 메서드를 이용해서 파일의 경로와 구분자 등의 정보를 출력하는 예제이다.     
절대경로는 파일시스템의 루트로부터 시작하는 파일의 전체 경로를 의미한다.      
OS에 따라 다르지만, 하나의 파일에 대해 둘 이상의 절대경로가 존재할 수 있다.    
현재 디렉토리를 의미하는'.'와 같은 기호나 링크를 포함하고 있는 경우가 이에 해당한다.    
그러나 정규경로는 기호나 링크 등을 포함하지 않는 유일한 경로를 의미한다.    
시스템속성 중에서 user.dir의 값을 확인하면 현재 프로그램이 실행 중인 디렉토리를 알 수 있다.    
그리고 우리가 OS의 시스템변수로 설정하는 classpath외에 sun.boot.class.path라는 시스템속성에    
기본적인 classpath가 있어서 기본적인 경로들은 이미 설정되어 있다. 그래서 처음 JDK설치 후 classpath를 따로 지정해주지 않아도 되는 것이다.     

```
boolean canRead()
읽을 수 있는 파일인지 검사한다.

boolean canWrite()
쓸 수 있는 파일인지 검사한다.

boolean canExecute()
실행할 수 있는 파일인지 검사한다.

int compareTo(File pathname)
주어진 파일 또는 디렉토리를 비교한다. 같으면 0을 반환하며, 다르면 1 또는 -1을 반환한다.

boolean exists()
파일이 존재하는지 검사한다.

boolean isAbsolute()
파일 또는 디렉토리가 절대경로명으로 지정되었는지 확인한다.

boolean isDirectory()
디렉토리인지 확인한다.

boolean isFile()
파일인지 확인한다.

boolean isHidden()
파일의 속성이 숨김인지 확인한다.
파일이 존재하지 않으면 false를 반환한다.

boolean createNewFile()
아무런 내용이 없는 새로운 파일을 생성한다.
생성하려는 파일이 이미 존재하면 생성되지 않는다.

static File createTempFile(String prefix, String suffix, File directory)
임시파일을 시스템의 지정된 디렉토리에 생성한다.

boolean delete()
파일을 삭제한다.

void deleteOnExit()
응용 프로그램 종료시 파일을 삭제한다.
주로 실행 시 작업에 사용된 임시파일을 삭제하는데 사용된다.

boolean equals(Object obj)
주어진 객체(주로 File인스턴스)가 같은 파일인지 비교한다.

long lastModified()
파일의 마지막으로 수정된 시간을 반환

long length()
파일의 크기를 반환한다.

String[] list()
디렉토리의 파일목록(디렉토리 포함)을 String배열로 반환한다.

String[] list(FilenameFilter filter)
File[] list(FilenameFilter filter)
FilenameFilter인스턴스에 구현된 조건에 맞는 파일을 String배열, File배열로 반환한다.

File[] listFiles()
File[] listFiles(FileFilter filter)
File[] listFiles(FilenameFilter f)
디렉토리의 파일목록(디렉토리 포함)을 File배열로 반환
filter가 지정된 경우에는 filter의 조건과 일치하는 파일만 반환

static File[] listRoots()
long getFreeSpace()
long getTotalSpace()
long getUsableSpace()
컴퓨터의 파일시스템의 root의 목록을 반환
get으로 시작하는 메서드들은 File이 root일 때 
비어있는 공간, 전체 공간, 사용가능한 공간을 바이트 단위로 변환

boolean mkdir()
boolean mkdirs()
파일에 지정된 경로로 디렉토리를 생성.
mkdirs는 필요하면 부모 디렉토리까지 생성

boolean renameTo(File dest)
지정된 파일dest로 이름을 변경

boolean setExecutable(boolean executable)
boolean setExecutable(boolean executable, boolean ownerOnly)
boolean setReadable(boolean readable)
boolean setReadable(boolean readable, boolean ownerOnly)
boolean setReadOnly()
boolean setWritable(boolean writable)
boolean setWritable(boolean writable, boolean ownerOnly)
파일의 속성을 변경한다.
ownerOnly가 true면 파일의 소유자만 해당 속성을 변경할 수 있다.

boolean setLastModified(long t)
파일의 마지막으로 수정된 시간을 지정된 시간t로 변경

Path toPath()
파일을 Path로 변환해서 반환

URI toURI()
파일을 URI로 변환해서 반환
```

```java
public class FileEx2 {
	public static void main(String[] args) {
		if(args.length!=1) {
			System.out.println("USAGE: java FileEx2 DIRECTORY");
			System.exit(0);
		}
		File f = new File(args[0]);
		
		if(!f.exists()||!f.isDirectory()) {
			System.out.println("유효하지않은 디렉토리입니다.");
			System.exit(0);
		}
		File[] files = f.listFiles();
		
		for(int i=0; i<files.length; i++) {
			String fileName = files[i].getName();
			System.out.println(
					files[i].isDirectory()?"["+fileName+"]":fileName);
		}
	}
}
```
지정한 디렉토리에 포함된 파일과 디렉토리의 목록을 보여주는 예제이다.

```java
public class FileEx3 {
	static int totalFiles = 0;
	static int totalDirs = 0;
	public static void main(String[] args) {
		if(args.length!=1) {
			System.out.println("USAGE:java FileEx3 DIRECTORY");
			System.exit(0);
		}
		
		File dir = new File(args[0]);
		if(!dir.exists()||!dir.isDirectory()) {
			System.out.println("유효하지 않은 디렉토리입니다.");
			System.exit(0);
		}
		printFileList(dir);
		
		System.out.println("총 "+totalFiles+" 개의 파일");
		System.out.println("총 "+totalDirs+" 개의 디렉토리");
	}
	
	public static void printFileList(File dir) {
		System.out.println(dir.getAbsolutePath()+" 디렉토리");
		File[] files = dir.listFiles();
		
		ArrayList subDir = new ArrayList();
		
		for(int i = 0; i<files.length; i++) {
			String filename = files[i].getName();
			
			if(files[i].isDirectory()) {
				filename = "["+filename+"]";
				subDir.add(i+"");
			}
			System.out.println(filename);
		}
		int dirNum = subDir.size();
		int fileNum = files.length-dirNum;
		
		totalFiles += fileNum;
		totalDirs += dirNum;
		
		System.out.println(fileNum+"개의 파일"+dirNum+"개의 디렉토리");
		System.out.println();
		
		for(int i =0; i<subDir.size(); i++) {
			int index = Integer.parseInt((String)subDir.get(i));
			printFileList(files[index]);
		}
	}
}
```
서브디렉토리와 그에 포함된 파일과 디렉토리의 목록까지 보여주는 예제이다.    
printFileList(File dir)는 디렉토리에 포함된 파일과 디렉토리의 목록을 출력하는 메서드인데 재귀호출을 이용하였다.    
먼저 파일의 목록을 출력하고 디렉토리인 경우 ArrayList에 담았다가 각 디렉토리에 대해 printFileList(File dir)를 재귀호출한다.    

```java
public class FileEx4 {

	public static void main(String[] args) {
		String currDir = System.getProperty("user.dir");
		File dir = new File(currDir);
		
		File[] files = dir.listFiles();
		
		for(int i =0; i<files.length; i++) {
			File f = files[i];
			String name = f.getName();
			SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm a");
			String attribute = "";
			String size = "";
			
			if(files[i].isDirectory()) {
				attribute = "DIR";
			}else {
				size=f.length()+"";
				attribute = f.canRead()?"R":" ";
				attribute += f.canWrite()? "W" : " ";
				attribute += f.isHidden()? "H": " ";
			}
			System.out.printf("%s %3s %6s %s\n",
					df.format(new Date(f.lastModified())),attribute, size, name);
		}
	}
}
```
현재 디렉토리에 속한 파일과 디렉토리의 이름과 크기 등 상세정보를 보여 주는 예제이다.
