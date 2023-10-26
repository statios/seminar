# 생성 디자인 패턴

생성 디자인 패턴 :

복잡한 생성 프로세스 캡슐화 → `객체의 생성` / `객체의 사용`을 각각 분리함

## 싱글턴 패턴 (1)

### 인스턴스를 단 하나만 생성할 수 있도록 하는 패턴

→ 한곳에서 관리되어야 하는 경우에 사용

e.g.

- 시스템 설정 정보 관리 클래스
- 고유 증분 ID 생성기 클래스

### **싱글턴을 구현할 때 고려해야 할 조건**

1. 생성자는 private해야 한다.
2. 생성될 때 스레드 safe 체크해야 한다.
3. 지연 로딩을 지원하는지 여부를 확인해야 한다.
4. getInstance() 함수의 성능이 충분해야 한다.

### 다양한 구현 방법들

1. 즉시 초기화하는 방법
    - 인스턴스가 사용되는 시점이 아니라 미리 생성되는 방식
    - 인스턴스 생성 프로세스가 스레드 세이프
    
    ```java
    public class IdGenerator {
    	private AtomicLong id = new AtomicLong(0);
    	private static final IdGenerator instance = new IdGenerator();
    	
    	private IdGenerator() {}
    	public static IdGenerator getInstance() {
    		return instance;
    	}
    
    	public long getId() {
    		return id.incrementAndGet();
    	}
    }
    ```
    
2. lazy 초기화하는 방법
    - 처음으로 사용될 때 인스턴스가 초기화됨
    - 초기화 할 때 처리해야할 작업이 많은 경우 → 성능에 영향
    
    ```java
    public class IdGenerator {
    	private AtomicLong id = new AtomicLong(0);
    	private static IdGenerator instance;
    
    	private IdGenerator() {}
    
    	public static synchronized IdGenerator getInstance() {
    		if (instance == null) {
    			instance = new IdGenerator();
    		}
    		return instance;
    	}
    
    	public long getId() {
    		return id.incrementAndGet();
    	}
    }
    ```
    
    <aside>
    💡 `synchronized` : **현재 데이터를 사용하고 있는 해당 스레드를 제외하고 나머지 스레드들은 데이터에 접근 할수 없도록 막는 개념 → getId() 호출하려고 할 때마다 getInstance()도 호출되어야 하므로 동시성이 제한됨.**
    
    </aside>
    
3. 이중 잠금
    - lazy 초기화 방식의 동시성 문제 해결
    
    ```java
    public class IdGenerator {
    	private AtomicLong id = new AtomicLong(0);
    	private static IdGenerator instance;
    
    	private IdGenerator() {}
    
    	public static IdGenerator getInstance() {
    		if (instance == null) {
    			synchronized(IdGenerator.class) { // 클래스 레벨의 잠금 처리
    				if (instance == null) {
    					instance = new IdGenerator();
    				}
    			}
    		}
    		return instance;
    	}
    
    	public long getId() {
    		return id.incrementAndGet();
    	}
    }
    ```
    
4. 홀더에 의한 초기화
    - lazy 초기화를 구현하면서 이중 잠금 방식보다 간단하게 구현
    
    ```java
    public class IdGenerator {
    	private AtomicLong id = new AtomicLong(0);
    	
    	private IdGenerator() {}
    
    	private static class SingletonHolder {
    		private static final IdGenerator instance = new IdGenerator();
    	}
    	
    	public static IdGenrator getInstance() {
    		return SingletonHolder.instance;
    	}
    
    	public long getId() {
    		return id.incrementAndGet();
    	}
    }
    ```
    

### Swift에서 자주 사용하는 Singleton 패턴

```swift
final class QandaAppController {

    static let shared = QandaAppController()

		private init() {}
}
```

- 콴다 프로젝트에서 싱글톤을 구현할 때 거의 이런 패턴을 따름
    - 초기화 구문을 `private`으로 지정하지 않은 것들이 꽤 보인다.
- swift의 global 변수는 `lazy` 키워드가 없어도 항상 lazy 초기화 됨 ([link](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/properties/#Global-and-Local-Variables))
- 인스턴스 lazy 변수는 thread-safe 하지않지만, 글로벌 변수의 경우에는 여러 스레드에서 동시에 접근하더라도 하나의 인스턴스만 생성되도록 보장.
- 즉 앱이 시작될 때 메모리 적재되기를 원한다면, 다른 방식을 고려해야 한다.

### 싱글턴 패턴의 단점

1. **클래스 간의 의존성을 감춘다** 
2. **확장성에 영향을 미친다**
3. **테스트 코드 짜기가 어려워진다** 
4. 생성자에 매개변수를 지원할 수 없다
    
    매개변수가 필요한 경우에 솔루션 → 별도로 정의한 config 타입의 전역변수를 의존하도록 하는 방법.
    
    ```swift
    struct QandaAppControllerConfig {
    	static var size: Int = 10
    }
    
    final class QandaAppController {
    
    		private var size: Int?
    
        static let shared: QandaAppController = {
    			let instance = QandaAppController
    			instance.size = QandaAppControllerConfig.size
    			return instance
    		}()
    
    		private init() {}
    }
    ```
    

### 싱글턴 패턴의 대안

클래스 인스턴스가 전역적으로 하나만 생성되도록 보장하면서 테스트 용이성과 확장성 문제를 해결 하기 위한 대안

→ 싱글톤 객체를 바로 사용하지 않고 외부에서 주입 받아서 사용

```swift
class ViewModel {

	let userRepository: UserRepository

	init(userRepository: UserRepository = UserRepositoryImpl.shared) {
	}
	
	func fetchUserName() -> String {
		return userRepository.getUserName // ✅
		// return UserRepositoryImpl.shared.getUserName ❎
	}
}
```

---

## 싱글턴 패턴 (2)

싱글턴 패턴은 하나의 인스턴스만 생성하는 것이 원칙이다.

이 유일성의 범위를 어디까지로 볼 것이냐에 따라 싱글턴 패턴을 다시 정의해볼 수 있다.

### 스레드 전용 싱글턴 패턴

특정 스레드에서만 유일한 인스턴스임을 보장하는 싱글턴 패턴

```java
public class IdGenerator {
	private AtomicLong id = new AtomicLong(0);
	private static funal ConcurrentHashMap<Long, IdGenerator> instances = new ConcurrentHashMap<>();

	private IdGenerator() {}

	public static IdGenerator getInstance() {
		Long currentThreadId = Thread.currentThread().getId();
		instances.putIfAbsent(currentThreadId, new IdGenerator());
		return instances.get(currentThreadId);
	}

	public long getId() {
		return id.incrementAndGet();
	}
}
```

### 클러스터 환경에서의 싱글턴 패턴

여러 프로세스 내에서 유일한 인스턴스임을 보장하는 싱글턴 패턴

1. 싱글턴 객체를 직렬화 하여 파일시스템에 저장
2. 프로세스 내에서 파일시스템에 접근하여 역직렬화하여 메모리로 읽어서 사용
3. 프로세스 내에서 사용을 마친 싱글턴 객체를 다시 직렬화하여 파일시스템에 저장하고 메모리에서 제거
4. 프로세스 내에서 사용하는 동안에 다른 프로세스에서 사용하지 못하도록 잠금

```java
public class IdGenerator {
	private AtomicLong id = new AtomicLong(0);
	private static IdGenerator instance;
	private static SharedObjectStorage storage = FileSharedObjectStorage(...);
	private static DistributedLock lock = new DistributedLock();

	private IdGenerator() {}

	public synchronized static IdGenerator getInstance() {
		if (instance == null) {
			lock.lock();
			instance = storage.load(IdGenerator.class);
		}
		return instance
	}

	public synchronized void freeInstance() {
		storage.save(this, IdGenerator.class);
		instance = null;
		lock.unlock();
	}

	public long getId() {
		return id.incrementAndGet();
	}
}
```

### 다중 인스턴스 패턴

싱글턴 패턴과 달리 여러 인스턴스를 생성할 수 있지만 그 수가 제한되는 패턴

## 팩토리 패턴 (1)

팩토리 패턴의 종류

- 단순 팩토리 패턴
- 팩토리 메서드 패턴
- 추상 팩토리 패턴

### 단순 팩토리 패턴

```java
public class ParserFactory {
	public static Parser createParser(String format) {
		Parser parser = null;
		if format == "json" {
			parser = JsonParser();
		} else if format == "xml" {
			parser = XMLParser();
		} else if format == "yaml" {
			parser = YamlParser();
		}
		return parser;
	}
}
```

- Naming tips
    - 클래스명 : SomethingFactory
    - 메서드명 : createSomething(), createInstance(), getInstance(), newInstance()
- 메모리 최적화를 위해서 매번 새로운 객체를 생성하서 반환하지 않고 팩토리 내부에서 캐시하여 구현할 수도 있다.

### 팩토리 메서드 패턴

단순 팩토리 패턴에서 if 흐름제어가 과도하게 사용되는 경우 개방 폐쇠 원칙에 위배

→ if 분기 대신 다형성을 사용

```java
public interface ParserFactory {
	Parser createParser();
}

public class JsonParserFactory implements ParserFactory {
	@override
	public Parser createParser() {
		return new JsonParser();
	}
}

public class XMLParserFactory implements ParserFactory {
	@override
	public Parser createParser() {
		return new XMLParser();
	}
}

public class YamlParserFactory implements ParserFactory {
	@override
	public Parser createParser() {
		return new YamlParser();
	}
}
```

### 추상 팩토리 패턴

matrix 관계를 가진 class object들을 생성하려고할 때 유용하게 사용할 수 있음.

|  | light mode | dark mode |
| --- | --- | --- |
| Button | LightButton | DarkButton |
| CheckBox | LightCheckBox | DarkCheckBox |

```swift
protocol UIFactory {
	func createDarkUI() -> UIView
	func createLightUI() -> UIView
}

class ButtonUIFactory: UIFactory {

	func createDarkUI() -> UIView {
		return DarkButton()
	}

	func createLightUI() -> UIView {
		return LightButton()
	}
}

class CheckBoxUIFactory: UIFactory {

	func createDarkUI() -> UIView {
		return DarkCheckBox()
	}

	func createLightUI() -> UIView {
		return LightCheckBox()
	}
}
```

### 팩토리 패턴의 적용 대상

- 객체 생성에 있어 if 분기가 있는 경우
    - 객체 생성 과정이 비교적 간단하다 → 단순 팩토리 패턴
    - 객체 생성 과정이 비교적 복잡하다 → 팩토리 메소드 패턴
    

## 팩토리 패턴(2)

### DI 컨테이너와 팩토리 패턴 비교

| 팩토리패턴 | DI컨테이너 |
| --- | --- |
| 작다 | 크다 |
| 하나의 타입에 대한 생성 담당 | 여러 타입에 대한 생성 담당 |
| 객체 생성 | 객체 생성 + 설정 분석 + 객체 수명 주기 관리 등 |

### DI 컨테이너 핵심 기능

1. 설정 분석
    - 런타임에 생성할 객체가 결정됨 → 무엇을 생성 할지 판단 기준이 ‘설정’
    - 객체 생성에 필요한 정보 또한 ‘설정’에 담긴다.
    
    ```java
    public class RateLimiter {
    	private RedisCounter redisCounter;
    	
    	public RateLimiter(RedisCounter redisCounter) {
    		this.redisCounter = redisCounter;
    	}
    
    	public void test() {
    		system.out.println("Hello World ")
    	}
    }
    
    public class RedisCounter {
    	private String ipAddress;
    	private int port;
    
    	public RedisCounter(String ipAddress, int Port) {
    		this.ipAddress = ipAddress;
    		this.port = port
    	}
    }
    ```
    
    RateLimiter 생성은 RedisCounter에 의존
    
    RedisCounter는 생성시 매개변수 필요
    
    → 생성시에 어떤 값을 사용할 지 설정 파일에서 표현
    
    ```xml
    <beans>
    	<bean id="rateLimiter" class="com.jpub.RateLimiter">
    		<constructor-arg ref="redisCounter"/>
    	</bean>
    
    	<bean id= "redisCounter" class="com.jpub.redisCounter">
    		<constructor-arg type="String" value="127.0.0.1">
    		<constructor-arg type="int" value=1234>
    	</bean>
    </beans>
    ```
    
2. 객체 생성
3. 객체 수명 주기 관리
    - 객체를 생성할 때 scope 옵션을 지정하여 생명 주기를 관리합니다.
        - scope=prototype : 매번 새로운 객체 생성 및 반환
        - scope=singleton : 미리 생성된 객체를 반환
    - lazy하게 생성할 지 즉시 생성할지 옵션 지원