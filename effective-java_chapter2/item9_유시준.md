# 주제
> try-finally보다 try-with-resources를 사용하라

# close
자바 라이브러리에는 close메서드를 사용해 닫아줘야 하는 자원이 많다.
Input/OutputStream 혹은 sql.Connection등이 그 예다. finalizer를 이용한 자원반납은 신뢰성이 부족하다. 전통적으로 자원반납에는 try-finally가 사용되었다.
```java
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
    try{
    	return br.readLine();
    }finally{
    	br.close();
    }
}
```
자원이 2개라면 더 복잡해진다.
```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
    try{
    	OutputStream out = new FileOutputStream(dst);
        try{
        	byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read(buf)) >=0)
            	out.write(buf,0,n);
        }finally{
        	out.close();
        }
	}finally{
    	in.close();
    }
}
```
위 두 코드예제에는 결점이 있다. 어떠한 이유로 try블럭에서 예외가 발생했는데 finally블럭에서도 예외가 발생한다면 finally블럭에서 발생한 예외가 try블럭에서 발생한 예외를 먹어버릴 것 이다.

[stack-overflow](https://stackoverflow.com/questions/48449093/what-is-wrong-with-this-java-puzzlers-piece-of-code)에서 알 수 있는 try-finally 단점

# try-with-resources
자바 7부터 try-with-resources를 사용할 수 있다. 이 구조를 사용하려면 AutoCloseable 인터페이스를 구현해야 한다. 자바 라이브러리에서는 이미 AutoCloseable을 구현해놓았다.
```java
static void copy(String src, String dst) throws IOException {
    try(InputStream in = new FileInputStream(src);
    	OutputStream out = new FileOutputStream(dst);){
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while((n = in.read(buf)) >=0)
	     	out.write(buf,0,n);
	}
}
```
위 코드를 아래와 같이 바꿀 수 있다. 그렇게 된다면 이점이 2가지 생긴다.
>
1. 코드가 간결해진다.
2. try내부 어디든 예외가 발생하더라도 in,out close가 모두 호출된다.
3. 처음 예외가 발생한 부분의 내용이 로깅되기 때문에 디버깅이 편하다.

# 결론
단순히 가독성이 좋아서 try-with-resources를 사용하는게 아니다.
