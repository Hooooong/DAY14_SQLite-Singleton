Android Programing
----------------------------------------------------
### 2017.09.21 8일차

#### 예제
____________________________________________________

- [SQLite 를 사용한 메모장 만들기](https://github.com/Hooooong/DAY14_SQLiteMemo)

#### 공부정리
____________________________________________________

##### __SQLite__

- SQLite 란?

    - SQLite란 오라클, MS-SQL, MySQL과 달리 소규모 DB에 사용되는 관계형 데이터베이스이다.

    - 데이터를 따로 백업하는 것이 아닌 파일 상태로 관리한다.

    - 기본적으로 이 공간은 다른 애플리케이션이 액세스할 수 없기 때문에 저장된 데이터는 안전하게 유지된다.

- SQLite 설정

    - SQLite 는 파일 단위로 관리하기에 경로는 `/data/data/패키지명/database/데이터베이스명` 이다.

    - Android 는 SQLite 접속을 쉽게 하기 위해 SQLiteOpenHelper 클래스를 재공한다.

    ```java
    public class DBHelper extends SQLiteOpenHelper {
    // 코드 작성
    }
    ```

    - SQLiteOpenHelper 는 기본적으로 생성자, onCreate(), onUpgrade() 메소드를 재정의해야 한다.

    ```java
    // DB name
    private static final String DB_NAME = "sqlite.db";
    // DB version
    private static final int DB_VERSION = 1;

    // 매개변수가 없는 Default 생성자가 없는 Class 를 상속받기 위해서는
    // Overloading 한 생성자를 호출해야 한다.
    // super(매개변수...)
    public DBHelper(Context context) {
        super(context, DB_NAME, null, DB_VERSION);

        // super 에서 넘겨받은 데이터베이스가 생성되어 있는지 확인한 후
        // 1. 없으면 onCreate 를 호출
        // 2. 있으면 version 을 체크해서 생성되어 있는 DB 보다 version 이 높으면 onUpgrade 를 호출한다.
    }
    ```

    - `생성자` 를 호출 시 DB 가 생성되어 있는지를 확인 한 후 없으면 `onCreate()`, 있으면 `onUpdate()` 를 실행한다.

    - __DB가 변경되면 `onCreate()`에 모든 히스토리를 반영해야 한다.__

    - __DB가 변경되면 `onUpgrade()`에서 version check 를 통해 version 별로 업데이트되는 내역 또한 반영 해야 한다.__

    ```java
    // DB를 새로 생성할 때 호출되는 함수
    @Override
    public void onCreate(SQLiteDatabase db) {
        // 최초 생성할 테이블 상의
        // settings.config

        // DB 가 업데이트 되면
        // 모든 히스토리가 쿼리에 반영되어 있어야 한다.
        String createDB = "CREATE TABLE `memo`                                \n" +
                          "( `id` INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT, \n" +
                          "  `title` TEXT, \n" +
                          "  `content` TEXT, \n" +
                          "  `nDate` TEXT \n" +
                          ")";
        // DB Query 실행
        db.execSQL(createDB);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // revision.config
        // App 을 update 를 하게 되면 코드들이 새로 엎어써지고, DB Version 을 확인해서

        // onUpgrade 를 실행하기위해 Alter Table 칼럼추가 를 하게 되면
        // onCreate 에도 반영이 되어야 하고

        // version check 를 통해 version 별로 업데이트 되는 내역 또한 반영이 되어야 한다.

        if(oldVersion < 2){
            //version 2
            // 쿼리
        }
    }
    ```

- SQLite 사용

    - `SQLiteDatabase` 객체를 사용하여 각종 Query 를 실행할 수 있다. `SQLiteDatabase` 는 읽기, 쓰기에 따라 getWritableDatabase(), getReadableDatabase() 메소드를 사용한다.

    ```java
    // 읽기
    SQLiteDatabase con = dbHelper.getWritableDatabase();
    // 쓰기
    SQLiteDatabase con = dbHelper.getWritableDatabase();
    ```

    - 실행 : 삽입, 수정, 삭제는 execSQL(실행할 쿼리문) 메소드를 사용한다.

    ```java
    con.execSQL(쿼리);
    ```

    - 실행 : 검색은 Cursor 객체를 활용하여 row 단위로 찾아온다.

    ![Cursor 사용](https://github.com/Hooooong/DAY14_SQLite-Singleton/blob/master/image/%EC%BA%A1%EC%B2%98.PNG)

    ```java
    // 2. 조작

    // 대소문자 구문을 꼭 해서 Column 명을 작성해야 한다.
    // 그렇지 않을 경우 getColumnIndex 에서 오류난다.
    Cursor cursor = con.rawQuery(query, null);
    while (cursor.moveToNext()) {
      for (String col : columns) {
        int index = cursor.getColumnIndex(col);
        switch (col) {
          case "id":
              // id 값 사용
              break;
          case "title":
              // title 값 사용
              break;
          case "content":
              // content 값 사용
              break;
          case "nDate":
              // nDate 값 사용
              break;
          }
      }  
    }
    ```

- 참조 : [SQL 데이터베이스에 데이터 저장](https://developer.android.com/training/basics/data-storage/databases.html?hl=ko#DbHelper)

##### __Singleton Pattern__

- Singleton Pattern 이란?

  > 소프트웨어 디자인 패턴에서 싱글턴 패턴(Singleton pattern)을 따르는 클래스는, 생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고 최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 리턴한다. 이와 같은 디자인 유형을 싱글턴 패턴이라고 한다. 주로 공통된 객체를 여러개 생성해서 사용하는 DBCP(DataBase Connection Pool)와 같은 상황에서 많이 사용된다.

  - 위 설명과 동일하게 DB에서 긴 시간이 걸리는 Connection 의 시간을 줄이기 위해 Singleton Pattern 을 사용한다.

  - Android 경우 Memory Pool 이 꽉차는 순간 가장 먼저하는것이 DB Connection 을 닫기 때문에 조심해서 작성해야 한다.

  ```java
  // DB가 가장 느린 구간은 첫 생성 시 Connection 할 때 가장 느리다.
  // 이 방법을 개선하기 위해 DB를 연결하게 하는 곳은 DBHelper 에서 Singleton Pattern 으로 작성한다.

  // 하지만 Memory Pool 이 꽉찰 경우 안드로이드는 가장 먼저하는것이 DB Connection 을 닫는 것을 한다.

  /**
   * Singleton Pattern
   */
  class Singleton{
      // App 전체에 new 가 하나만 되어야 한다.
      private static Singleton singleton;

      private Singleton(){}

      public static Singleton getInstance(){
          if(singleton == null){
              return new Singleton();
          }
          return singleton;
      }
  }
  ```

- 참조 : [Singleton Pattern](https://ko.wikipedia.org/wiki/%EC%8B%B1%EA%B8%80%ED%84%B4_%ED%8C%A8%ED%84%B4)
