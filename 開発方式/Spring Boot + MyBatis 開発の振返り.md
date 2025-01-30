# Spring Boot + MyBatis 開発の振返り

## 課題

### ① インターフェース駆動開発の過剰適用
サービスクラス、リポジトリクラスに必ずインターフェースを設け、実装クラスをそれに基づいて作成するという方式を適応していた。  
インターフェースは抽象化による柔軟性を提供するが、用途が限られた小規模なサービスやリポジトリにまで適用すると、無駄なコードや工数が増加することがある。

### ➁ 過度な分離による冗長化
以下の層に厳密に分ける方式を全てに適応していたが、層を厳密に分離する方式では単純な処理（例: 入力値の保存）の実装が複雑化することがある。
1.	コントローラ層: ユーザーインターフェース
2.	サービス層: ビジネスロジック
3.	リポジトリ層: データアクセス
また、**MyBatis を使用するため、データアクセス時には `@Mapper` アノテーションを付与した Mapper インターフェースを作成していた。**  
さらに、**リポジトリクラスを必ず作成するルール** を設けていたため、以下のような呼び出し構成になっていた。

- **コントローラ層（Controller）**  
  ↓  
- **サービス層（Service）**  
  ↓  
- **リポジトリ層（Repository）**  
  ↓  
- **Mapper インターフェース（MyBatis）**  

この方式を一律に適用したことで、**リポジトリクラスと Mapper インターフェースが 1 対 1 になり、リポジトリクラスが単に Mapper のメソッドを呼び出し、その結果をそのまま返すだけのケースが多発**した。  
そのため、**リポジトリクラスを作成する必要があるケースと、不要なケースを適切に使い分けるべき**だと考えた。

## 提案

### ① インターフェースの導入基準
インターフェースは以下のケースに限定して導入する。

モックやスタブを使用するユニットテストが必須の場合
  - サービスやリポジトリ層の依存をモック化してテストしたい時に有効。例えば、外部依存を排除し、純粋にビジネスロジックのテストが可能となる。(テスト実行の高速化)

拡張性が求められる場合
  - ビジネス要件の変更が頻繁に発生する箇所や複数の実装が必要な時に有効。
```
public interface PaymentService {
    void processPayment(PaymentRequest request);
}

@Service
public class CreditCardPaymentService implements PaymentService {
    @Override
    public void processPayment(PaymentRequest request) {
        // クレジットカード支払い処理
    }
}

@Service
public class PaypayPaymentService implements PaymentService {
    @Override
    public void processPayment(PaymentRequest request) {
        // PayPay支払い処理
    }
}
```

チーム分担の必要がある場合
  - 先にインターフェースを作る人、中身を実装する人などに分けるケースに有効。

## ➁ 層の省略/適用の基準

単純な CRUD 操作の場合、サービス層を省略
```
@RestController
@RequestMapping("/users")
public class UserController {
    private final UserMapper userMapper;

    public UserController(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        return this.userMapper.findById(id);
    }
}
```
ビジネスロジックが複雑な場合はサービス層を導入
例えば、複数リポジトリの統合処理している場合など。
```
@Service
public class AService {
    private final B1Repository b1Repository;
    private final B2Repository b2Repository;

    public AService(B1Repository b1Repository, B2Repository b2Repository) {
        this.b1Repository = b1Repository;
        this.b2Repository = b2Repository;
    }

    public void updateA() {
        this.b1Repository.update();
        this.b2Repository.update();
    }
}
```
例外処理やトランザクション管理が必要な場合はサービス層を導入
```
@Service
public class UserService {
    private final UserMapper userMapper;

    public UserService(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Transactional
    public User createUser(User user) {
        if (user.getName() == null || user.getName().isEmpty()) {
            throw new IllegalArgumentException("ユーザー名は必須です");
        }
        return userMapper.save(user);
    }
}
```
リポジトリクラスが不要な場合
  - リポジトリクラスと Mapper インターフェースが 1 対 1 となるとき
  - リポジトリクラスが Mapper の単純なラッパーにしかなっていないとき
```
@Service
public class UserService {
    private final UserMapper userMapper;

    public UserService(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    public User getUserById(Long id) {
        return userMapper.findById(id);
    }
}
```
リポジトリクラスが必要な場合
  - 複数のデータソースを統合する
  - SQL クエリの前処理・後処理が必要
  - 独自のロジックを追加する
```
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User findById(Long id);
}

@Repository
public class UserRepository {
    private final UserMapper userMapper;

    public UserRepository(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    public User getUserWithProcessing(Long id) {
        User user = userMapper.findById(id);
        // 何らかのデータ加工処理を追加
        user.setProcessed(true);
        return user;
    }
}
```