# [Spring 공부 부분]

**스프링 나무소리 유튜브 강의 영상 사이트: https://www.youtube.com/watch?v=XL39DGSsYCs&list=PLOSNUO27qFbsW_JuXmzrFxPw7qzPOFfQs&index=1**
```
ClubMapStore은 Map형식의 저장소에 club데이터를 저장하고 읽어오는 작업들을 구현했다.

서비스와 스토어의 역할에서 헷갈리지말아야할점은,
스토어는 데이터베이스에 직접 접근하는 메소드들을 구현해놓은것이고,
서비스는 클라이언트가 원하는 요청 내용만을 담아 스토어의 어떠한 메소드를 불러올지 바탕 메소드들을 구현해놓고, 즉, 서비스는 스토어의 데이터베이스 접근 메소드들을 호출시키는 용도의 중간 처리자 메소드 역할인것이다.
즉, 서비스는 스토어의 데이터베이스 접근 메소드를 호출해주는 중간 다리 역할이라는 사실을 잊지말자.

Map클래스같은 스토어 영역에서는 CRUD처럼 데이터를 불러오거나 저장하거나 지우거나 수정하거나 하는 등등의 단순한 작업을 위주로 하여 복잡한 로직을 수행하지는 않는 클래스이고,
Service같은 서비스 영역에서는 스토어에 데이터 처리 요청 역할뿐만 아니라, 데이터를 로직적으로 계산해야하는 여러가지 절차들이 있을때 그러한 코드들을 함께 적어줄수있는 클래스이다.

추후에 main메소드에서 Travelclub에 대한 새로운 데이터가 생성이 되면,
그걸 가지고 ClubServiceLogic으로 해당 데이터를 보내주는 처리해달라는 요청하는 역할을 하고,
그럼으로써 ClubServiceLogic은 그 데이터를 Map에 저장할 수 있게끔 하기위해서 ClubMapStore에게 전달하여 ClubMapStore 메소드를 호출한다.
즉, ClubServiceLogic에서 ClubMapStore를 알아야하는 관계가 성립된다.

ClubServiceLogic <->> ClubStore 인터페이스 <->> ClubMapStore

IoC를 구현하는 방법이 DI라서 보통 IoC DI 라고 이야기한다.

ClubMapStore를 ClubServiceLogic이 알게하고 사용할수있게끔 하려면,
Spring IoC 컨테이너로 하여금 ClubMapStore가 bean 객체라는것을 알 수 있도록 등록해주어야한다.
즉, 먼저 ClubMapStore를 bean으로 등록해야한다.
그리고 이렇게 등록된 ClubMapStore는 ClubServiceLogic이 참조해서 사용할 수 있도록 의존관계(DI)를 주입해줘야 한다.
그다음 ClubServiceLogic도 bean으로 등록해주고, 등록하면서 생성자의 파라미터로 인터페이스 객체를 넣어 ClubMapStore를 연결시켜 DI시켜준다.

public class ClubServiceLogic implements ClubService {

    private ClubStore clubStore;  // ClubStore 인터페이스 타입의 필드(변수) 선언.

    public ClubServiceLogic(ClubStore clubStore) {  // ClubServiceLogic 은 ClubMapStore 을 알아야하는 관계이기때문에 이렇게 적어준다.
        this.clubStore = clubStore;  // 등호(=) 왼쪽의 this.clubStore은 private ClubStore clubStore;의 clubStore 이고, 등호(=) 오른쪽의 clubStore은 public ClubServiceLogic(ClubStore clubStore)의 매개변수인 clubStore 이다.
        // 아마 이렇게 적은 이유는, ClubServiceLogic과 ClubMapStore은 ClubStore 인터페이스를 사이에 두고 느슨한 결합을 유지해야하기때문에, DI에 ClubStore 인터페이스를 사용한것같다.
        // 이렇게하면, 장점은 ClubStore 인터페이스를 사이에 두고있어, ClubServiceLogic 입장에서는 ClubStore 인터페이스를 참조하고있는 객체가 ClubMapStore던지 아니면 다른 추후에 변경된 DB든지간에 알아서 DI주입이 될테니 상관없어진다는 것이다.
        // 즉, 나중에 DB 종류를 변경할때 다른 코드는 건드리지않고 그대로 써도 되기때문에 편해진다.
    }
}

만약 의존관계 DI 방식중,
직접 Bean 등록방식말고 
컴포넌트 스캔 방식을 사용할 경우,
Gradle 기준으로는 ClubServiceLogic 의 클래스위에 @Service를, ClubServiceLogic 의 의존관계 연결 메소드 위에는 @Autowired를 적어주어야한다. 그리고 ClubMapStore 클래스위에는 @Repository를 적어주어야한다.
Maven 기준으로는 applicationCotext.xml 에다가 <context:component-scan base-package="스캔 적용 패키지 전체범위"/>를 적어주고, @Autowired 없이 ClubServiceLogic 의 클래스위에 @Service만을, ClubMapStore 클래스위에는 @Repository를 적어주어야한다. 아닌가? 아마 Gradle처럼 @Autowired 적어주어도 되는듯하다.
참고로 @Service 같은 어노테이션 오른쪽에 @Service("설정하고싶은Bean이름") 이렇게 적으면 Bean이름을 직접 원하는대로 지을수있다.

이상하게도
직접 Bean 등록 방식으로 DI해주면, Bean등록이름이 clubServiceLogic, clubStore 이고,
컴포넌트 스캔 방식으로 Bean 등록해서 DI해주면, Bean등록이름이 clubServiceLogic, clubMapStore 이다.
아, 내가 그렇게 적어서그런가보다. 원래는 clubServiceLogic, clubMapStore 가 맞는듯하다.
이래서 나중에 데이터베이스 저장소 종류를 변경할수도있을때는 컴포넌트 스캔 방식보다는 직접 Bean 등록방식 으로 Service와 Store을 DI 하는걸 추천하는가보다.

sdo는 'service domain object'의 약자이다.
그리고 sdo 패키지 안에있는 Cdo 클래스들은 'creative domain object'의 약자이다.
즉, create메소드?처럼 create될때 사용할 damain object를 따로 Cdo로 만들어준것이다.
즉, create 될때 필요한 데이터(필드)들을 별도의 domain object로 나누어 적어둔것이 Cdo인것이다.
TravelClub이 생성될때 id값은 Entity에 의해 자동생성부여되고, foundationTime은 현재시각으로 자동생성부여되므로, 막상 Travelclub 클래스 내에서 직접 필요한 필드(변수)는 id와 foundationTime을 제외한, name과 intro 변수밖에 없다.
그러므로, TravelClub이 생성될때 사용되는 object인 TravelClubCdo 클래스에는 name과 intro 변수만 선언되어있는것이다.
즉, 새로운 TravelClub 데이터에는 name과 intro 정보밖에 필요하지않다는 뜻이다.
예를들어 ClubServiceLogic 클래스에서 Club등록(즉, create 역할) 용도인 registerClub 메소드를 사용할때에, 매개변수로 Cdo인 TravelClubCdo 가 들어가있는것이다.
참고로 intro 는 '소개'라는 뜻의 변수로 지은것이다.

< Maven에서, 테스트 케이스에 새로운 Club 등록 테스트 코드 작성과 자세한 설명 >
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        // 윗줄은 Maven에서 사용하는코드 (Gradle 아님.)
        TravelClubCdo clubCdo = new TravelClubCdo("FirstTravelClub", "Test TravelClub");
        // 이제 밑의코드줄부터 ClubService를 통해서 객체를 넘겨주어야한다.
        // ClubServiceLogic 클래스의 registerClub 메소드를 호출하기위해 Service 관련 Bean을 먼저 스프링컨테이너에서 찾아와야한다.
        ClubService clubService = context.getBean("clubServiceLogic", ClubService.class);  // getBean의 매개변수: ("등록해둔 Bean 이름", 찾아오는 타입)
        // 보다시피 ClubServiceLogic 클래스를 직접사용하지않고, ClubService 인터페이스를 사용함으로써 느슨한 결합을 유지해준다. 즉, 인터페이스를 사이에 두고 의존관계를 형성한것이다.
        String clubId = clubService.registerClub(clubCdo);
        // 과정: TravelCdo객체 생성하여 값넣어서, 새로 등록될 순수 필드값정보만 담고있는 Cdo 데이터 전달
        // -> 스프링컨테이너에 이미 등록되어있는 Service 빈을 갖고와서 Store 과의 의존관계 확인
        // -> ClubServiceLogic 클래스의 인스턴스 객체인 clubService로 registerClub 메소드를 실행하여, Cdo객체에 등록된 순수한 필드값 정보를 넘겨준다
        // -> registerClub 메소드 안에서 TravelClub 클래스의 TravelClub(String name, String intro)로 객체생성할때 Cdo 필드정보가 넘어가고
        // -> 결국 TravelClubCdo 순수 필드 정보가 새로운 TravelClub을 생성하는 과정에 필드정보가 넘어가서, TravelClub 클래스로 새로운 Club 등록 역할을 하게됨.
        // 여기서 주의할점은 create 될때 필요한 데이터(필드)들을 별도의 domain object로 나누어 적어둔것이 Cdo라는 점을 잊지말아야한다.
        // 즉 전체적인 과정은, TravelCdo의 순수 멤버필드정보가 TravelClub으로 새로운 Club의 정보를 생성하는데 기여하고, 그렇게 생성된 정보객체가 Store에 create 메소드로 저장을 요청한다.
        // 과정 정리: TravelCdo의 순수 멤버필드정보 --registerClub 메소드에 대입해서 서비스 실행--> TravelClub으로 Club 정보 생성 -> Store의 create 메소드로 Map에 저장후 해당 Club 정보를 데이터베이스에 저장하길 요청
        // 위 과정 속의 구조 정리:
        // [일반 과정] TravelCdo의 순수 멤버필드정보
        // [서비스 구조 과정] --registerClub 메소드에 대입해서 서비스 실행--> TravelClub으로 Club 정보 생성 ->
        // [스토어 구조로 데이터베이스접근 과정] Store의 create 메소드로 Map에 저장후 해당 Club 정보를 데이터베이스에 저장하길 요청
        // 마지막 과정으로, Store의 create 메소드가 실행되고 id를 리턴return해줌. 그러면 그 id값을 clubId 변수에 할당함.
        System.out.println("Test에서 등록될 새로운 TravelClub의 ID : " + clubId);

< Gradle에서, SpringConfig 클래스에 직접 빈 등록방법으로 Bean 등록하여 DI 형성하였을때, 사용가능한 새로운 Club 등록 테스트 코드 >
    @Test
    void NewClubTest_springconfigDI() {
        TravelClubCdo clubCdo = new TravelClubCdo("FirstTravelClub", "Test TravelClub");
        // 이제 밑의코드줄부터 ClubService를 통해서 객체를 넘겨주어야한다.
        // ClubServiceLogic 클래스의 registerClub 메소드를 호출하기위해 Service 관련 Bean을 먼저 스프링컨테이너에서 찾아와야한다.
        ClubService clubService = ac.getBean("clubServiceLogic", ClubService.class);  // getBean의 매개변수: ("등록해둔 Bean 이름", 찾아오는 타입)
        // 보다시피 ClubServiceLogic 클래스를 직접사용하지않고, ClubService 인터페이스를 사용함으로써 느슨한 결합을 유지해준다. 즉, 인터페이스를 사이에 두고 의존관계를 형성한것이다.
        String clubId = clubService.registerClub(clubCdo);
        // 과정: TravelCdo객체 생성하여 값넣어서, 새로 등록될 순수 필드값정보만 담고있는 Cdo 데이터 전달
        // -> 스프링컨테이너에 이미 등록되어있는 Service 빈을 갖고와서 Store 과의 의존관계 확인
        // -> ClubServiceLogic 클래스의 인스턴스 객체인 clubService로 registerClub 메소드를 실행하여, Cdo객체에 등록된 순수한 필드값 정보를 넘겨준다
        // -> registerClub 메소드 안에서 TravelClub 클래스의 TravelClub(String name, String intro)로 객체생성할때 Cdo 필드정보가 넘어가고
        // -> 결국 TravelClubCdo 순수 필드 정보가 새로운 TravelClub을 생성하는 과정에 필드정보가 넘어가서, TravelClub 클래스로 새로운 Club 등록 역할을 하게됨.
        // 여기서 주의할점은 create 될때 필요한 데이터(필드)들을 별도의 domain object로 나누어 적어둔것이 Cdo라는 점을 잊지말아야한다.
        // 즉 전체적인 과정은, TravelCdo의 순수 멤버필드정보가 TravelClub으로 새로운 Club의 정보를 생성하는데 기여하고, 그렇게 생성된 정보객체가 Store에 create 메소드로 저장을 요청한다.
        // 과정 정리: TravelCdo의 순수 멤버필드정보 --registerClub 메소드에 대입해서 서비스 실행--> TravelClub으로 Club 정보 생성 -> Store의 create 메소드로 Map에 저장후 해당 Club 정보를 데이터베이스에 저장하길 요청
        // 위 과정 속의 구조 정리:
        // [일반 과정] TravelCdo의 순수 멤버필드정보
        // [서비스 구조 과정] --registerClub 메소드에 대입해서 서비스 실행--> TravelClub으로 Club 정보 생성 ->
        // [스토어 구조로 데이터베이스접근 과정] Store의 create 메소드로 Map에 저장후 해당 Club 정보를 데이터베이스에 저장하길 요청
        // 마지막 과정으로, Store의 create 메소드가 실행되고 id를 리턴return해줌. 그러면 그 id값을 clubId 변수에 할당함.
        System.out.println("Test에서 등록될 새로운 TravelClub의 ID : " + clubId);
    }

< Gradle에서, SpringConfig 클래스 없이 @Repository, @Service, @Autowired 로 Bean 등록하여 DI 형성하였을때, 사용가능한 새로운 Club 등록 테스트 코드 >
    @Test
    void NewClubTest_annotationDI() {  // 근데 이건 내가 직접 만든 코드라 약간 불확실함. 테스트 메소드 정상실행되긴함.
        TravelClubCdo clubCdo = new TravelClubCdo("FirstTravelClub", "Test TravelClub");
        ClubStore clubStore = new ClubMapStore();
        ClubService clubService = new ClubServiceLogic(clubStore);
        String clubId = clubService.registerClub(clubCdo);
        System.out.println("Test에서 등록될 새로운 TravelClub의 ID : " + clubId);
    }

< Controller과 Service과 Store 레이어 간의 교류 과정 >
클라이언트 요청 -> Controller -> Service -> Store
Controller --ClubService 인터페이스를 사이에 두고 느슨한 결합--> ClubServiceLogic --ClubStore 인터페이스를 사이에 두고 느슨한 결합--> ClubMapStore
컨트롤러 --서비스 인터페이스 사이에 두고--> 서비스 --스토어 인터페이스 사이에 두고--> 스토어

html 같은 뷰파일없이 바로 json으로 데이터를 보내 출력할거면, @Controller와 @ResponseBody가 결합된 @RestController 어노테이션을 컨트롤러 클래스 위에 적어두고,
html 같은 뷰파일 사용시 컨트롤러 클래스 위에 @Controller 어노테이션을 적어둔다.

< ClubController 클래스 >
@RestController  // 데이터가 담겨오는 방식은 @RequestParam("") 사용해서 url주소에 따라오느냐, 아니면 @RequestBody를 사용해서 http request의 body에 담겨오느냐인데, @RestController 이므로, json 사용으로 @RequestBody를 적어주자.
// html 같은 뷰파일없이 바로 json으로 데이터를 보내 출력할거면, @Controller와 @ResponseBody가 결합된 @RestController 어노테이션을 컨트롤러 클래스 위에 적어두고,
// html 같은 뷰파일 사용시 컨트롤러 클래스 위에 @Controller 어노테이션을 적어둔다.
public class ClubController {

    // ClubController --ClubService 인터페이스를 사이에 두고 느슨한 결합--> ClubServiceLogic --ClubStore 인터페이스를 사이에 두고 느슨한 결합--> ClubMapStore
    private ClubService clubService;
    @Autowired
    public ClubController(ClubService clubService) {  // 인터페이스 사이에 두고 느슨한 결합 생성자 주입 DI
        this.clubService = clubService;
    }

    @PostMapping("/club") // 등록하는 컨트롤러 메소드이므로, POST방식으로 데이터를 받아와야 가능한 메소드이다.
    public String register(@RequestBody TravelClubCdo travelClubCdo) {  // 클라이언트로부터 요청이 올때, TravelClubCdo 데이터가 오며 가장 먼저 컨트롤러로 위임될것이고,
                                                           // TravelClubCdo의 객체를 파라미터로 데이터로 받는다면
        return clubService.registerClub(travelClubCdo);  // Controller 레이어에서 Service 레이어로 전달
                                                         // (추후에 서비스에서 clubStore.create() 메소드를 호출하며, Store 레이어로 전달하면, 최종적으로 DB에 등록 완료.)
                                                         // 즉, 레이어 전달 과정이 클라이언트요청->Controller->Service->Store 인것이다.
    }

}

public class ClubController {}  // ClubServiceLogic과 연관된 메소드들의 컨트롤러 버전의 메소드를 구현하면 된다.

travel-grd h2 데이터베이스
jdbc:h2:tcp://localhost/~/travelgrd 이걸로 연결

hello-spring h2 데이터베이스
jdbc:h2:tcp://localhost/~/test 이걸로 연결

아마도 h2를 소켓으로 실행시키고 나서 스프링 프레임워크에서 서버를 돌려야지 될듯하다.
그리고 h2 로그인.

< TravelClubJpo 클래스 >
// 도메인 객체를 바로 Entity로 매핑할 수 있지만, 임피던스 불일치(기존 관계형 데이터베이스의 SQL과 프로그래밍 언어 사이에 데이터 구조, 기능 등의 차이로 발생하는 충돌) 등의 이유로, 별도의 매핑 클래스 (Jpo)를 둔다.
// 이 @Entity를 선언해주어야, 여기 클래스 테이블을 h2데이터베이스 같은 DB에서 JPA로 매핑을 해주고 DB 테이블을 자동으로 만들어줄수있다.
@Entity  // 이것은 JPA가 관리하는 엔티티이다 라는 것이다. 매핑클래스를 관계형 데이터베이스 테이블로 매핑할때 사용한다.
@Getter
@Setter
@NoArgsConstructor  // @NoArgsConstructor 어노테이션은 파라미터(매개변수)가 없는 기본 생성자 메소드를 생성해준다. TravelClubJpo(){} 메소드 말이다.
                    // 보통 엔티티에 @NoArgsConstructor를 같이 사용하는데, 그 이유는 @Entity는 JPA가 관리하는 엔티티라는 뜻인데, java의 ORM 기술인 JPA는 기본 스펙상 기본 생성자를 요구하기때문이다.
                    // 하지만 사실 @Entity 어노테이션이 내부적으로 기본 생성자를 자동으로 만들어주기때문에, @NoArgsConstructor( access = AccessLevel.PROTECTED) 이런식처럼 조건을 추가로 붙이지 않는한, 보통 @Entity 붙어있으면 @NoArgsConstructor을 생략하여도된다.
                    // 참고로 단, 초기 값이 필요한 final 필드가 있을 경우, 컴파일 에러이다. 그럴때는 @NoArgsConstructor(force=true)를 하면, 컴파일 에러를 내지 않고 0/false/null로 초기화해준다.
@Table(name="TRAVEL_CLUB")  // @Table(~)없이 @Entity 어노테이션만 선언했을때 테이블 이름은 클래스 이름인 'TRAVEL_CLUB_JPO'가 된다. 그러므로 @Table 어노테이션으로 직접 테이블 이름을 지어주면 그 테이블명으로 테이블이 생성된다.
public class TravelClubJpo {  // 이 클래스는 데이터베이스 엔티티 객체라고 생각하면 된다고 한다.

    @Id  // 해당 컬럼을 PK로 설정한다.
    private String id;
    private String name;
    private String intro;
    private long foundationTime;

    public TravelClubJpo(TravelClub travelClub) {  // TravelClub 객체인 domain객체를 가져와서 TravelClubJpo 객체인 Jpo객체로 바꿔줌으로써, 데이터베이스 엔티티 객체로 사용할수있게하는 용도이다.
//        this.id = travelClub.getId();  // this.~ 이것은 TravelClubJpo 클래스의 필드들이다.
//        this.name = travelClub.getName();  // this.~ 이것은 TravelClubJpo 클래스의 필드들이다.
//        this.intro = travelClub.getIntro();  // this.~ 이것은 TravelClubJpo 클래스의 필드들이다.
//        this.foundationTime = travelClub.getFoundationTime();  // this.~ 이것은 TravelClubJpo 클래스의 필드들이다.

        BeanUtils.copyProperties(travelClub, this);  // 위의 this코드 4줄을 이렇개 한줄로 압축시킬수있다. travelClub 객체의 프로퍼티 복사해서 this인 TravelClubJpo 객체로 전달.
    }

    public TravelClub toDomain() {  // 위의 TravelClubJpoJpo 메소드와는 반대로, 객체를 domain객체로 바꿔주는 메소드이다.
        TravelClub travelClub = new TravelClub(this.name, this.intro);  // TravelClub 클래스의 TravelClub(String name, String intro) 메소드를 사용하였다.  // this.~ 이것은 TravelClubJpo 클래스의 필드들이다.
        travelClub.setId(this.id);  // TravelClub 클래스의 부모인 Entity 클래스의 Getter 어노테이션덕에 setId 사용가능.  // this.~ 이것은 TravelClubJpo 클래스의 필드들이다.
        travelClub.setFoundationTime(this.foundationTime);  // TravelClub 클래스의 Setter 어노테이션덕에 setFoundationTime 메소드를 사용가능.  // this.~ 이것은 TravelClubJpo 클래스의 필드들이다.
        return travelClub;
    }
}  // 여기까지만 만든것이면, 아직 spring data jpa를 적용한것이 아니라, 단순히 jpa hibernate 기술만 가지고 사용해본것이다.

< ClubJpaStore 클래스 일부 >
@Repository
// ClupMapStore 대신 ClubJpaStore 로 저장소를 변경할거라,
// ClupMapStore 클래스의 @Repository 어노테이션 제거해서 빈등록 안되게 막고,
// 대신 ClubJpaStore 클래스에 @Repository 어노테이션 붙여줘서 빈등록하게함.
public class ClubJpaStore implements ClubStore {
    // 참고로 엔터티 매니저(EntityManager)는 엔터티를 저장하는 메모리상의 데이터베이스라고 생각하면 된다. 엔터티를 저장하고 수정하고 삭제하고 조회하는 등 엔터티와 관련된 모든일을 한다.
    // 원래는 EntityManager라는 객체(예를들어 private final EntityManager em;)를 이용한
    // em.persist() 이나 em.find() 와 같은 영속성 메소드를 사용하여, 순수 jpa를 통해 데이터를 넣고 쓰고 사용할 수 있지만,
    // 순수 jpa에서 EntityManager를 사용하던 이러한 작업들을, 이제는 더욱 추상화하여 더 쉽게 쓸 수 있도록 해주는 방법인 spring data jpa 가 나와서 전자의 방법 대신 이걸 사용하면 된다.
    // 즉, 일일이 jpa에서 제공하는 EntityManager를 이용해서 데이터를 읽고 쓰는게 아니라, EntityManager에 대한 관리조차도 spring data 에게 맡기면 되게 되었다.
}

ClubController --ClubService 인터페이스를 사이에 두고 느슨한 결합--> ClubServiceLogic --ClubStore 인터페이스를 사이에 두고 느슨한 결합--> ClubJpaStore --Jpa에서 제공하는 JpaRepository 인터페이스를 상속하는 ClubRepository 인터페이스를 사이에 두고 느슨한 결합--> 결국은 Jpa를 통해 Spring Data Jpa 의 DB에 접근 성공
// 참고로 그러면 JpaRepository 인터페이스에 기본적으로 등록되어있는 CRUD Jpa 접근 메소드와, ClubRepository 인터페이스에 직접 등록하여 정의한 메소드를 Jpa 접근 메소드로 활용할 수 있다.

< ClubRepository 인터페이스 >
public interface ClubRepository extends JpaRepository<TravelClubJpo, String> {  // 인터페이스가 인터페이스를 상속받을때에는 implements가 아닌, extends를 사용한다. 그리고 콤마(,)로 구분하여 다중 상속이 가능하다.
                                                                                // <TravelClubJpo, String>는 <@Entity가 적혀있는 TravelClubJpo 클래스의 DB 엔티티, TravelClubJpo 엔티티의 pk id의 자료형인 String> 이다.  // 자료형 <class T,ID 식별자 pk 자료형>

    List<TravelClubJpo> findAllByName(String name);  // 만약 findAllByName가 아닌, findByName 이었다면 반환자료형이 List가 아니었을것이다.
    // name으로 검색해서 찾아서 그 이름이 같은 모든 Club데이터들을 모두 가지고 올것이기 때문에, 메소드의 반환자료형이 List인것이고,
    // TravelClubJpo 가 담긴 리스트를 반환해서 가져와야하기때문에, findAllByName 메소드의 반환자료형은 List<TravelClubJpo> 가 되는것이다.

    // JpaRepository안에 매우 기본적이고 공통적인 CRUD등이 전부 구현되어 있기 때문에,
    // retrieveAll() 안에 쓰인 findAll(), create()와 update() 안에 쓰인 save(), retrieve() 안에 쓰인 findById()는
    // 이미 JpaRepository 인터페이스에 있는 Spring Data Jpa 관련 CRUD 메소드 안에 이미 들어있어서 따로 구현해줄 필요없이 그저 Jpa 메소드를 갖다쓰면 되는데,
    // 하지만, retrieveByName() 안에 쓰일 name값으로 데이터를 찾는 메소드처럼 JpaRepository에 없는 특별한 경우에 대해서는 구현되어 있기 어렵다. (모든 시스템이 다르기 때문에)
    // 이유는 find의 검색조건이 여러가지가 될 수 있기에 우리가 지정한 변수인 name에 대해서는 JpaRepository에 따로 구현되어있지 않는것이다.
    // 그래서 findAllByName()은 직접 구현해주어야한다. 참고로 이 findAllByName() 메소드는 우리가 메소드명을 직접 지어준것이다.
    // 결국 JpaRepository 인터페이스에 구현되어있지 않은 CRUD 메소드를, 여기 ClubRepository 인터페이스에서 따로 만들어주고 이 메소드를, ClubJpaStore 클래스에서 Jpa DB 접근 메소드로 불러내어 직접 사용할 수 있게 되는 것이다.

    // 참고로 이처럼 직접 JPA 메소드를 정의하여 만들어줄때는, JPA(Java Persistence API) 자동 생성 쿼리 메소드의 명명 규칙에 따라서 만들어주면 된다.
}

```
