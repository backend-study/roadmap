# JDK 8
* JDK 1.8 이전과 이후는 람다, 스트림, 옵셔널 사용이 가능하냐의 여부에 따라 크게 나뉜다. 이는 자바가 함수형 프로그래밍을 가능하게 하고 ```NullPointerException```으로부터 자유롭게 하는, 모던 자바 형태의 프로그래밍을 가능하도록 한다.
## Lambda 람다
### 개요
* 람다는 함수를 변수처럼 사용하는 표현식이다. 함수를 파라미터로 취급한다는 발상은 함수를 일급 객체로 여기겠다는 의미인데, 파라미터로 다른 메소드의 인자로 함수를 넘길 수 있다. 이는 자바스크립트와 같이 함수형 프로그래밍을 자바에서도 구현할 수 있도록 해 준다. JDK 7 버전 이하에서 구현했던 익명 함수와 쓰임이 동일하다.

### 예제
> 코드스쿼드 자바지기님의 자료를 참고했습니다.

* Collection의 모든 값을 출력한다고 가정해보자.
* 아래 예시는 for-each 문을 이용해 구현한 것이다.
```javascript=
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

for (int number : numbers) {
    System.out.println(number);
}
```
* 람다가 없던 시절에는, forEach 구문을 활용하여 내부에 익명함수를 선언하여 구현했다.
```javascript=
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

numbers.forEach(new Consumer<Integer>() {
    public void accept(Integer value) {
        System.out.println(value);
    }
});
```
* 람다를 활용하면 다음과 같이 구현할 수 있다.
```javascript=
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

numbers.forEach((Integer value) -> System.out.println(value));
numbers.forEach(value -> System.out.println(value)); // Type 추론이 가능해 Type 생략 가능
numbers.forEach(System.out::println); // :: 연산자 활용 가능
= numbers.forEach(x -> System.out.println(x));
```
## Stream 스트림
### 개요
* 스트림은  데이터의 흐름(stream)이다. 이전에는 Collection 인스턴스의 내용을 변경하거나 이를 활용하기 위해서는 for문을 돌면서 일일이 요소를 꺼내서 작업해야 했다. 하지만 스트림이 등장하면서부터 컬렉션 인스턴스 단위에서 함수 여러 개를 조합해서 원하는 결과를 필터링할 수 있게 되었다. 이를 '함수형' 으로 프로그래밍한다고 부른다.
### 실전 개념
* 스트림에는 시작 연산, 중간 연산, 최종 연산이 있다. 
    * 스트림을 만들어서 데이터를 가공하는 과정에서 스트림을 생성하는 연산. 주로 ```stream()``` 이 있음.
    * 데이터를 필터링하거나 가공하는 연산이 있음. 주로 ```map```, ```filter```, ```sorted``` 등을 사용함. 
    * 최종적으로 연산 받은 것을 모아서 사용자에게 반환해주는 형태를 명시하는 연산이 있음. 주로 ```stream``` 컬렉션을 사용함.
* IntStream을 사용하면 index와 range에 따른 스트림 생성이 가능하다. ```range```와 ```rangeClosed```를 자유자재로 사용하면 범위만큼 for문을 도는 효과가 발생한다.
```javascript=
private Lines drawLines(Height height) {
	int playerCount = players.getSize();
	int heightCount = height.getHeightValue();

	return IntStream.range(0, heightCount)
		.mapToObj(index -> Line.ofLine(playerCount, paintingStrategy))
		.collect(collectingAndThen(toList(), Lines::ofLines));
	}
```
* ```map```은 특정 요소를 해당하는 값으로 변환해준다. 모든 요소를 일일이 순회한다는 특징이 있다.
* 아래 예제에서는 Repository가 반환한 컬렉션을 일일이 순회하면서 DTO를 생성하고, 이를 하나의 List로 담고 있다.

```javascript=
public List<RoomResponseDto> findAll() {
	Long userId = userService.findUserIdByToken();
	return roomRepository.findAll().stream()
		.map(room -> RoomResponseDto.create(room, isUserBookmarkedRoom(room.getId(), userId)))
		.collect(Collectors.toList());
	}
```
* ```filter```는 boolean값으로 true인 요소들만 다음 스트림으로 넘겨준다. 
    * ```findAny()``` 또는 ```findFirst()```와 같은 최종 연산을 활용하면 필터링된 요소를 바로 반환받을 수 있다.
    * ```anyMatch()```, ```allMatch()``` 또는 ```noneMatch()```를 활용하면 필터링 연산에 대한 boolean 값을 조건에 따라 반환받을 수 있다.
* 아래 예시에서는 ```Player``` 라는 사용자 객체를 일급 컬렉션으로 들고 있는 ```Players``` 내부의 리스트를 순회하면서 특정한 인덱스 조건에 맞는 ```filter``` 연산을 수행하고, 이후 가장 먼저 해당하는 요소를 반환하고 있다. 이후에는 ```map``` 이라는 중간 연산을 추가로 수행하여 최종 연산에서 반환받은 요소에 한번 더 수정을 가하고 있다. 
```javascript=
public Player playerPrizeMapFactory(Prizes prizes, Prize prize) {
	return players.stream()
		.filter(player -> player.getCurrentPosition().getPosition() == prizes.prizeIndex(prize))
		.findFirst()
		.map(player -> player.updatePrize(prize))
		.orElseThrow(() -> new IllegalArgumentException("no user"));
	}
```
* 아래 예시는 ```allMatch``` 연산을 통해 모든 원소가 Null 값임을 검증하는 데 사용되고 있다. 이 때에는 ```filter``` 중간 연산은 생략할 수 있다.
```javascript=
public static boolean isEmpty(List list) {
	return list.stream().allMatch(Objects::isNull);
}
```
* collect 라이브러리를 멋들어지게 사용하면 스트림의 활용도가 배가 된다. 특정한 리스트에서 스트림을 생성해서 데이터를 가공하고 난 뒤, 이를 어떻게 반환할 지 직접 사용자가 지정할 수 있기 때문이다. 
* 다음의 예시를 보자. ```String``` 인스턴스에서 ```split()``` 메소드를 불러와서 Arrays로 만든 다음에, 이를 스트림으로 생성하여 작업하고 있다. ```map``` 연산을 통하여 하나하나의 원소마다 Player 객체를 생성하고 있고, 이를 최종적으로 모아서 리스트로 던져준다. 
* 재미있는 점은 ```collect``` 연산 안에 ```collectingAndThen```이라는 메소드가 있다는 점인데, 이는 map으로 넘어오는 모든 원소를 모아서 하나의 리스트로 만든 다음, 해당 리스트를 사용하여 ```Players```라는 객체의 인스턴스를 한 방에 생성하겠다는 선언이다. 즉, collect 연산 내부에서 최종 반환값까지 모두 사용자 지정이 가능하다.
```javascript=
//Players.java
public static Players ofPlayers(List<Player> players) {
    return new Player(players);
}
```

```javascript=
//PlayerService.java
String nameString = ...
return Arrays.stream(nameString.split(PLAYER_NAME_DELIMITER))
	.map(Player::ofPlayer)
	.collect(collectingAndThen(toList(), Players::ofPlayers));
//Players가 반환된다.
```
* ```forEach```와 ```peek``` 은 모두 원소를 일일이 순회한다는 특징이 있다. 이 둘의 차이점은 ```peek()``` 은 중간 처리 메소드이고, ```forEach()```는 최종 처리 메소드라는 점이다.

```javascript=
public static void main(String[] args){
        int[] intArr = {1, 2, 3, 4, 5};

        // 최종처리 메소드가 없으면 동작하지 않음.
        Arrays.stream(intArr)
            .filter(a -> a%2 == 0)
            .peek(System.out::println); // peek은 중간 처리 메소드(intermediate)

        // 최종처리 메소드(Terminal) sum()이 존재하므로 정상 작동한다.
        Arrays.stream(intArr)
            .filter(a -> a%2 == 0)
            .peek(System.out::println)
            .sum();

        // forEach는 최종처리 메소드(Terminal) 이므로 아래는 정상 동작한다.
        Arrays.stream(intArr)
            .filter(a -> a%2 == 0)
            .forEach(System.out::println);
}
// https://cornswrold.tistory.com/299
```
* 스트림 생성은 비싸다. 하지만 함수형 프로그래밍을 가능하게 하는 API이다. 따라서 적절한 활용이 필요하다고 생각되는데, 하나의 메소드 내부에 하나의 스트림만 생성하도록 하자.

## Optional 옵셔널
### 개요
* 자바에서는 ```NullPointerException``` 만큼 짜증나는 것이 없다. 그나마 자바 14부터는 NPE가 어느 부분에서 발생되었는 지 알려주는 기능이 추가되었지만, 우리가 주로 사용하는 자바 8에는 그러한 기능이 없다.
* 옵셔널을 사용하면 Null이 도출될 수 있는 객체를 한 번 감싸는 행위를 통해(wrapping) Null일 때, Null이 아닐 때 각각의 경우에 따른 별도의 분기처리가 가능하다.
* 옵셔널을 사용하면 어느 부분에서 객체 값이 Null이 나왔는 지 알려주므로, 에러 슈팅에 수월하다.
* 박성철님의 다음 슬라이드 자료를 참고하면 자바에서 안전하게 NULL 값을 처리할 수 있는 방법에 대해 살펴볼 수 있다. [링크](https://www.slideshare.net/gyumee/null-142590829)
* 자바 8 이전에는 학교, 교실, 선생님, 과목이라는 클래스가 있을 때 이 학교의 교실의 선생님의 과목을 반환하는 코드를 작성하면 다음과 같이 된다. 전혀 Null-Safe하지 않다.
```javascript=
school.getClassRoom().getTeacher().getSubject().getSubjectName();
```
* Null 처리를 추가해보자. ```우웩!```
```javascript=
if(school != null) {
    ClassRoom classRoom = school.getClassRoom();
    if(classRoom != null) {
        Teacher teacher = classRoom.getTeacher();
        if(teacher != null) {
            Subject subject = teacher.getSubject();
            if(subject != null) {
                String subjectName = subject.getSubjectName();
                return subjectName;
            }
        }
    }
}
return null;
```
* 자바 8에서 도입된 옵셔널을 사용해보자. 편안하다.
```javascript=
Optional.ofNullable(school).map(School::getClassRoom)    //Optional<School>
                           .map(ClassRoom::getTeacher)   //Optional<ClassRoom>
                           .map(Teacher::getSubject)     //Optional<Teacher>
                           .map(Subject::getSubjectName) //Optional<Subject>
                           .orElse(null);
```
### 실전 개념
* ```Optional.of()```와 ```Optional.ofNullable()```로 객체를 감쌀 수 있다. of는 인자로서 null값을 받지 않는다는 것이고 ofNullable은 null값을 허용한다는 것이다.

```javascript=
// 안 좋음
return Optional.of(member.getEmail());  // member의 email이 null이면 NPE 발생

// 좋음
return Optional.ofNullable(member.getEmail());

// 안 좋음
return Optional.ofNullable("READY");

// 좋음
return Optional.of("READY");

// http://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/
```
* Optional에서 반환값이 없으면 기본값을 지정할 수 있다.
```javascript=
String lastWordInLexicon = max(words).orElse("단어 없음..");
```
* Optional에서 반환값이 없으면 오류를 던질 수 있다.
```javascript=
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```
* ```Optional.empty```를 사용하면 빈 값을 반환하고자 할 때 null이 반환값이 아니도록 처리할 수 있다. 이는 NPE를 방지하는 좋은 방법이다.
* ```map``` 연산을 활용할 수 있다.
    * Optional로 받은 객체를 원래 객체로 돌리기 전에, 스트림의 ```map```처럼 중간 연산을 할 수 있다. 이렇게 될 경우 ```map```으로 연산을 한 객체를 반환받을 수 있다.
```javascript=
private static int printScore(int sum, Frame frame) {
	Optional<Score> totalScore = frame.calculateFrameTotalScore();
	if (!totalScore.isPresent()) {
		System.out.print("        |");
		return sum;
	}
	sum += totalScore.map(Score::getScore).get();
	System.out.printf("%s| ", sum);
	return sum;
}
```
* Optional은 값을 포장하고 다시 풀고, 값이 없을 때 대체하는 값을 넣는 등의 오버헤드가 있으므로, 무분별하게, 적절하지 않게 사용된다면 성능 저하가 있을 수 있기 때문에 사용에 신중해야 한다.
* 또한 Optional은 검사 예외와 취지가 비슷하기 때문에 간혹 구현 실수로 비검사 예외를 던지거나 null을 반환 한다면 사용자가 그 사실을 인지하지 못해 런타임에서 예상치 못한 장애로 발전할 수도 있다.