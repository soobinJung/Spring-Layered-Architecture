# Spring-Layered-Architecture

## Spring Layered architecture 의 계층에 대해 이론적으로 설명합니다.

### Presentation Layer (표현 계층)
사용자 인터페이스(UI)와 사용자 경험(UX)을 관리합니다. 사용자의 요청을 받아들이고, 사용자에게 정보를 시각적으로 전달하는 역할을 합니다.

특징
- 인증(Authentication)과 JSON 변환 같은 기능을 포함하여, 사용자가 시스템과 상호작용할 때 필요한 데이터 형식 변환과 사용자 신원 확인을 담당합니다.

### Business Layer (비즈니스 계층)
애플리케이션의 핵심 비즈니스 로직을 구현합니다. 데이터의 유효성 검사(Validation), 비즈니스 규칙의 실행, 사용자 권한 확인(Authorization) 등을 포함합니다.

특징
- 애플리케이션의 기능과 관련된 모든 규칙과 계산이 이 계층에서 처리됩니다. 사용자의 요청에 따라 적절한 비즈니스 로직을 수행하고 결과를 표현 계층으로 전달합니다.

### Persistence Layer (지속성 계층)
데이터의 영구 저장 및 검색을 관리합니다. 이 계층은 데이터를 데이터베이스에 저장하거나 데이터베이스에서 데이터를 검색하는 로직을 담당합니다.

특징
- 데이터베이스와 직접적인 상호작용을 수행하지만, 저장되는 데이터의 형식이나 구조에 대한 결정은 아니며, 단지 데이터를 영구적으로 저장하고 검색하는 방법을 정의합니다.

### Database Layer (데이터베이스 계층)
실제 데이터베이스 관리 시스템(DBMS)을 통해 데이터를 저장하고 관리합니다. 이 계층은 데이터의 구조(스키마 설계), 인덱싱, 트랜잭션 관리 등 데이터베이스의 물리적인 측면을 담당합니다.

특징
- 모든 데이터는 이 계층에서 관리되며, 상위 계층은 이 계층을 통해 데이터에 접근합니다. 데이터의 실제 저장 위치, 최적화, 백업 등의 관리도 포함됩니다.


## Spring Layered architecture 의 계층에 대해 java code 로 설명합니다.

### Presentation Layer (표현 계층)
표현 계층에서는 UserController가 사용자 요청을 처리하고, JSON 형태로 데이터를 반환하거나 받습니다. 인증도 이곳에서 처리되는데, 사용자 자격증명을 받아 서비스 계층으로 전달합니다.
```
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getUserById(@PathVariable Long id) {
        UserDto user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }

    @PostMapping("/authenticate")
    public ResponseEntity<String> authenticate(@RequestBody UserCredentials credentials) {
        boolean isAuthenticated = userService.authenticate(credentials);
        if (isAuthenticated) {
            return ResponseEntity.ok("User authenticated successfully");
        } else {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Authentication failed");
        }
    }
}
```

### Business Layer (비즈니스 계층)
비즈니스 계층에서는 UserService가 사용자의 신원을 확인하고 비즈니스 로직을 실행합니다. 사용자 정보를 가져오거나 인증하는 등의 작업을 수행합니다.
```
@Service
public class UserService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public UserService(UserRepository userRepository, PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    public UserDto getUserById(Long id) {
        return userRepository.findById(id)
                .map(UserMapper::toDto)
                .orElseThrow(() -> new UserNotFoundException("User not found with id: " + id));
    }

    public boolean authenticate(UserCredentials credentials) {
        return userRepository.findByUsername(credentials.getUsername())
                .map(user -> passwordEncoder.matches(credentials.getPassword(), user.getPassword()))
                .orElse(false);
    }
}
```

### Persistence Layer (지속성 계층)
지속성 계층에서는 UserRepository 인터페이스가 데이터베이스와의 상호작용을 담당합니다.

```
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

### Database Layer (데이터베이스 계층)
데이터베이스 계층은 실제 SQL 테이블이나 NoSQL 컬렉션과 직접 대응합니다. Spring 에서는 이를 직접 다루지 않지만, JPA 엔티티와 데이터베이스 스키마를 매핑하여 설명할 수 있습니다.
```
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String password;
    // getters and setters
}
```
