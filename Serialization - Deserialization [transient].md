## Serialization - Deserialization [transient]

### 1. Serialization과 Deserialization이 무엇인가요?

#### 1-1. Serialization

Serialization은 한글로 직역하면 직렬화라는 뜻으로,
직렬화는 네트워크 상에서 데이터를 전달할 때 byte 단위로 끊어서 보내기 위해서 사용하는 기법으로
자바에서는 자바의 객체를 byte 단위로 변경하여 보낼 때 사용된다.

#### 1-2. Deserialization

Deserialization은 한글로 직역하면 역직렬화라는 뜻으로,
이전에 직렬화를 통해 네트워크에서 데이터를 주고 받았다면
그 데이터를 받았을 때 그 객체를 원본상태로 되돌리는 기법을 역직렬화라고 한다.

### 2. 자바에서는 이를 어떻게 구현하나요?

자바에서는 보통 ObjectInputStream, ObjectOutputStream을 통해서 구현한다.
보통 파일 사이의 데이터 교환을 사용하므로 FileInputStream과 FileOutputStream도 같이 사용한다.
그럼 예제를 살펴보는 것이 이해에 도움이 되므로 예제를 먼저 살펴보도록 하겠다.

```java
/* User.java */

import java.io.Serializable;

public class User implements Serializable {
	private String name;
	private String id;
	private String pw;
	private String email;
	
	public User(String name, String id, String pw, String email) {
		this.name = name;
		this.id = id;
		this.pw = pw;
		this.email = email;
	}
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getPw() {
		return pw;
	}
	public void setPw(String pw) {
		this.pw = pw;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}

	@Override
	public String toString() {
        String formatString = "User{ name : %s, id : %s, pw : %s, email : %s }"; 
		return String.format(formatString, this.name, this.id, this.pw, this.email);
	}
}
```

위의 User 클래스가 파일간의 데이터 전송시 전달될 데이터이다.
그런데 User 클래스를 보면 기존의 데이터를 담은 VO 형식과는 다른 점이 있다.
바로 Serializable 인터페이스를 구현한다는 것인데
자바에서 직렬화를 통해 데이터 전송을 할 때에는 그 데이터가 Serializable 인터페이스를 구현하고 있어야 한다.
따라서 자바에서는 데이터 전송을 할 때 그 데이터가 모두 직렬화할 수 있는지 검사하고
그 객체의 프로퍼티도 모두 직렬화를 할 수 있는지 검사한다.
검사하는 기준은 다음과 같다.

-   Serializable를 구현하고 있는 객체이다.
-   byte, short, int, long, float, double, boolean, char와 같은 기본 자료형은 상관 없다.
-   String이나 Vector와 같은 경우에는 이미 Serializable를 구현하고 있다.

다음은 이 User 객체를 이용하여 데이터를 전송하는 SerializationDeserialization 클래스이다.

```java
/* SerialzationDeserialization.java */

/* import문 생략 */

public class SerializationDeserialization {
	private final String FILE_NAME;
	
	private FileOutputStream fos = null;
	private ObjectOutputStream out = null;
	
	private FileInputStream fis = null;
	private ObjectInputStream in = null;
	
	public SerializationDeserialization(String FILE_NAME) {
		this.FILE_NAME = FILE_NAME;
		try {
			fos = new FileOutputStream(FILE_NAME);
			out = new ObjectOutputStream(fos);
			
			fis = new FileInputStream(FILE_NAME);
			in = new ObjectInputStream(fis);
		} catch(Exception e) {
			e.printStackTrace();
		}
	}
	
	public void serializing(User user1, User user2) {
		List<User> list = new ArrayList<>();
		list.add(user1);
		list.add(user2);
		
		try {
			out.writeObject(user1);
			out.writeObject(user2);
			out.writeObject(list);
		} catch(Exception e) {
			e.printStackTrace();
		}
	}
	
	public void deserializing() {
		User user1 = null;
		User user2 = null;
		List<User> list = null;
		try {
			user1 = (User) in.readObject();
			user2 = (User) in.readObject();
			list = (List<User>) in.readObject();
		} catch(Exception e) {
			e.printStackTrace();
		}
		
		System.out.println(user1);
		System.out.println(user2);
		System.out.println(list);
	}
	
	public void close() throws IOException {
		out.close();
		fos.close();
		
		in.close();
		fis.close();
	}
}
```

생성자에서는 FileInputStream, FileOutputStream, ObjectInputStream, ObjectOutputStream을
초기화하고 전송할 파일의 이름을 정하며
serialization() 메소드는 ObjectOutputStream 파일 전송 객체를 이용하여 test.txt에 직렬화한
User 객체 둘, 그리고 그 User 객체 둘을 담은 List를 전송한다.
deserialization() 메소드는 직렬화를 통해 test.txt에 작성된 객체를
역직렬화하여 다시 자바 객체로 되돌리는 역할을 한다.

그럼 이 SerializationDeserialization 클래스를 사용하는 main 클래스를 작성해보도록 하겠다.

```java
/* MainClass.java */

public class MainClass {
	public static void main(String[] args) throws IOException, ClassNotFoundException {
		User user1 = new User("이진혁", "jinhyeok", "aaaaa", "aaaaa@gmail.com");
		User user2 = new User("김대웅", "daewoong", "bbbbb", "bbbbb@naver.com");
		User user3 = new User("손정우", "jeongwoo", "ccccc", "ccccc@hanmail.net");
		User user4 = new User("유시온", "yoozion", "ddddd", "ddddd@github.com");
		
		SerializationDeserialization sd = new SerializationDeserialization("test.txt");
		
		System.out.println("-----------------------------------------"
                           + "-----------------------------------------------");
		sd.serializing(user1, user2);
		sd.deserializing();
		System.out.println("-----------------------------------------"
                           + "-----------------------------------------------");
		sd.serializing(user3, user4);
		sd.deserializing();
		System.out.println("-----------------------------------------"
                           + "-----------------------------------------------");
		
		sd.close();
	}
}

/*
----------------------------------------------------------------------------------------
User{ name : 이진혁, id : jinhyeok, pw : null, email : aaaaa@gmail.com }
User{ name : 김대웅, id : daewoong, pw : null, email : bbbbb@naver.com }
[User{ name : 이진혁, id : jinhyeok, pw : null, email : aaaaa@gmail.com }, User{ name : 김대웅, id : daewoong, pw : null, email : bbbbb@naver.com }]
----------------------------------------------------------------------------------------
User{ name : 손정우, id : jeongwoo, pw : null, email : ccccc@hanmail.net }
User{ name : 유시온, id : yoozion, pw : null, email : ddddd@github.com }
[User{ name : 손정우, id : jeongwoo, pw : null, email : ccccc@hanmail.net }, User{ name : 유시온, id : yoozion, pw : null, email : ddddd@github.com }]
----------------------------------------------------------------------------------------
*/
```

총 두 번의 직렬화, 역직렬화를 하는데
첫 번째 직렬화를 할 때 test.txt에 User 객체 둘, User 객체 둘을 담은 리스트가 직렬화된 채로 담기고
역직렬화를 통해 총 세 객체를 받아와서 출력하고
두 번째 직렬화를 할 때 test.txt에 있었던 값에 이어서 작성하게 된다.
물론 새로 컴파일하게 되면 모든 기록이 다시 쓰여지게 된다.

그리고 직렬화와 역직렬화를 하면 직렬화를 한 순서대로 역직렬화가 되는 것을 알 수 있다.
user1 -> user2 -> list 순으로 직렬화를 했다면 역직렬화를 했을 경우
user1 -> user2 -> list 순으로 역직렬화가 되어 받아오는 것을 알 수 있다.

### 3. transient 키워드 사용하기

위의 예제 코드에서는 User 객체를 전송하는데 아무 보안도 하지 않은 채 보내게 됩니다.
물론 ObjectInput(Output)Stream에서 자체 보안이 있긴 하지만
굳이 보내고 싶지 않은 프로퍼티가 존재할 수 있다.
그럴 경우 사용하는 것이 transient 키워드이다.
transient 키워드를 사용할 경우 그 프로퍼티의 위치에 null이 삽입이 된다.
User 객체와 같은 경우 비밀번호를 숨기고 싶을 때가 있을 것이다.
이럴 때 pw 프로퍼티에 transient 키워드를 작성하면 pw가 null로 전송되고 null로 받아진다.
하지만 이를 사용할 경우에는 이 값이 null이라는 것을 인지하고 사용해야 한다.
(데이터가 전송되었을 때 pw를 사용하는 경우가 많아서 그렇다.)

따라서 pw라는 값을 숨기고 싶다면 다음과 같이 User 클래스를 고치면 된다.

```java
/* User.java */

import java.io.Serializable;

public class User implements Serializable {
	private String name;
	private String id;
	private transient String pw;
	private String email;
	
    // ...
}
```

### 4. serialVersionUID

serialVersionUID는 직렬화를 통해 byte 단위로 끊어서 보낸 데이터가 정상적으로 역직렬화가 되었는지,
전달 중 중간에 문제가 생기지는 않았는지를 확인하는 고유한 값이다.

기본적으로 명시해주지 않으면 컴파일러가 대신 다음과 같은 상수를 클래스에 삽입한다.

```java
private static final long serialVersionUID = "456789876543456789";
```

물론 숫자는 컴파일러가 계산하여 넣는 것이다.

하지만 자바에서는 이를 직접 넣어주는 것을 권한다.
따라서 serialVersionUID를 작성하는 것이 좋다.