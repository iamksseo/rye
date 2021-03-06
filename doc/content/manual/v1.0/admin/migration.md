데이터베이스 내보내기(unload)
=============================

데이터베이스를 언로드/로드하는 목적은 다음과 같다.

-   데이터베이스 볼륨을 재구성하여 데이터베이스 재구축
-   시스템이 다른 환경에서 마이그레이션 수행
-   버전이 다른 DBMS에서 마이그레이션 수행

<!-- -->

    rye unloaddb [options] database_name

**rye unloaddb**가 생성하는 파일은 다음과 같다.

-   스키마 파일(*database-name***\_schema**): 해당 데이터베이스에 정의된 스키마 정보를 포함하는 파일이다.
-   객체 파일(*database-name***\_objects**): 해당 데이터베이스에 포함된 인스턴스 정보를 포함하는 파일이다.
-   인덱스 파일(*database-name***\_indexes**): 해당 데이터베이스에 정의된 인덱스 정보를 포함하는 파일이다.
-   트리거 파일(*database-name***\_trigger**): 해당 데이터베이스에 정의된 트리거 정보를 포함하는 파일이다. 만약 데이터를 로딩하는 동안 트리거가 구동되는 것을 원치 않는다면, 데이터 로딩을 완료한 후에 트리거 정의를 로딩하면 된다.

이러한 스키마, 객체, 인덱스, 트리거 파일은 같은 디렉터리에 생성된다.

다음은 **rye unloaddb** 에서 사용하는 \[options\]이다.

데이터베이스 가져오기(load)
===========================

데이터베이스 로드는 다음과 같은 경우에 **rye loaddb** 유틸리티를 이용하여 수행된다.

-   이전 버전의 Rye 데이터베이스를 새로운 버전의 데이터베이스로 마이그레이션하는 경우
-   타 DBMS의 데이터베이스를 Rye 데이터베이스로 마이그레이션하는 경우
-   **INSERT** 구문 실행보다 빠른 성능으로 대용량 데이터를 입력하는 경우

일반적으로 **rye loaddb** 유틸리티는 **rye unloaddb** 유틸리티가 생성한 파일(스키마 정의 파일, 객체 입력 파일, 인덱스 정의 파일)을 사용한다. :

    rye loaddb [options] database_name

**입력 파일**

-   스키마 파일(*database-name***\_schema**): 언로드 작업에 의해 생성된 파일로서, 데이터베이스에 정의된 스키마 정보를 포함하는 파일이다.
-   객체 파일(*database-name***\_objects**): 언로드 작업에 의해 생성된 파일로서, 데이터베이스에 포함된 레코드 정보를 포함하는 파일이다.
-   인덱스 파일(*database-name***\_indexes**): 언로드 작업에 의해 생성된 파일로서, 데이터베이스에 정의된 인덱스 정보를 포함하는 파일이다.
-   트리거 파일(*database-name***\_trigger**): 언로드 작업에 의해 생성된 파일로서, 데이터베이스에 정의된 트리거 정보를 포함하는 파일이다.
-   사용자 정의 객체 파일(*user\_defined\_object\_file*) : 대용량 데이터 입력을 위해 사용자가 테이블 형식으로 작성한 입력 파일이다(howtowrite-loadfile 참고).

다음은 **rye loaddb** 에서 사용하는 \[options\]이다.

**--no-logging** 옵션을 사용하면 **loaddb** 를 수행하면서 트랜잭션 로그를 저장하지 않으므로 데이터 파일을 빠르게 로드할 수 있다. 그러나 로드 도중 파일 형식이 잘못되거나 시스템이 다운되는 등의 문제가 발생했을 때 데이터를 복구할 수 없으므로 데이터베이스를 새로 구축해야 한다. 즉, 데이터를 복구할 필요가 없는 새로운 데이터베이스를 구축하는 경우를 제외하고는 사용하지 않도록 주의한다. 이 옵션을 사용하면 고유 위반 등의 오류를 검사하지 않아 기본 키에 값이 중복되는 경우 등이 발생할 수 있으므로, 사용 시 이러한 문제를 반드시 감안해야 한다.

가져오기용 파일 작성 방법
=========================

**rye loaddb** 유틸리티에서 사용되는 객체 입력 파일을 직접 작성하여 사용하면 데이터베이스에 대량의 데이터를 보다 신속하게 추가할 수 있다. 객체 입력 파일은 간단한 테이블 모양의 형식으로 구성되며 주석, 명령 라인, 데이터 라인으로 이루어진 텍스트 파일이다.

주석
----

Rye에서는 주석은 두 개의 연속된 하이픈(--)을 이용하여 처리한다. :

    -- This is a comment!

명령 라인
---------

명령 라인은 퍼센트(%) 문자로 시작하며, 명령어로는 클래스를 정의하는 **%class** 명령어와, 클래스 식별을 위해 사용하는 별칭(alias)이나 식별자(identifier)를 정의하는 **%id** 명령어가 있다.

### 클래스에 식별자 부여

**%id** 를 이용하여 참조 관계에 있는 클래스에 식별자를 부여할 수 있다. :

    %id class_name class_id
    class_name:
        identifier
    class_id:
        integer

**%id** 명령어에 의해 명시된 *class\_name* 은 해당 데이터베이스에 정의된 클래스 이름이며, *class\_id* 는 객체 참조를 위해 부여한 숫자형 식별자를 의미한다.

    %id employee 2
    %idoffice 22
    %id project 23
    %id phone 24

### 클래스 및 속성 명시

**%class** 명령어를 이용하여 데이터가 로딩될 클래스(테이블) 및 속성(칼럼)을 명시하며, 명시된 속성의 순서에 따라 데이터 라인이 작성되어야 한다. **rye loaddb** 유틸리티를 실행할 때 **-t** 옵션으로 클래스 이름을 제공하는 경우에는 데이터 파일에 클래스 및 속성을 명시하지 않아도 된다. 단, 데이터가 작성되는 순서는 클래스 생성 시의 속성 순서를 따라야 한다. :

    %class class_name ( attr_name [attr_name... ] )

데이터를 로딩하고자 하는 데이터베이스에는 이미 스키마가 정의되어 있어야 한다.

**%class** 명령어에 의해 명시된 *class\_name* 은 해당 데이터베이스에 정의된 클래스 이름이며, *attr\_name* 는 정의된 속성 이름을 의미한다.

다음은 *employee* 라는 클래스에 데이터를 입력하기 위하여 **%class** 명령으로 클래스 및 3개의 속성을 명시한 예제이다. **%class** 명령 다음에 나오는 데이터 라인에서는 3개의 데이터가 입력되어야 하며, 이는 conf-reference-relation 을 참조한다. :

    %class employee (name age department)

데이터 라인
-----------

데이터 라인은 **%class** 명령 라인 다음에 위치하며, 입력되는 데이터는 **%class** 명령에 의해 명시된 클래스 속성과 타입이 일치해야 한다. 만약, 명시된 속성과 타입이 일치하지 않으면 데이터 로드 작업은 중지된다.

또한, 각각의 속성에 대응되는 데이터는 적어도 하나의 공백에 의해 분리되어야 하며, 한 라인에 작성되는 것이 원칙이다. 다만, 입력되는 데이터가 많은 경우에는 첫 번째 데이터 라인의 맨 마지막 데이터 다음에 플러스 기호(+)를 명시하여 다음 라인에 데이터를 연속적으로 입력할 수 있다. 이 때, 맨 마지막 데이터와 플러스 기호 사이에는 공백이 허용되지 않음을 유의한다.

### 인스턴스 입력

다음과 같이 명시된 클래스 속성과 타입이 일치하는 인스턴스를 입력할 수 있다. 각각의 데이터는 적어도 하나의 공백에 의해 구분된다.

    %class employee (name)
    'jordan' 
    'james'  
    'garnett'
    'malone'

### 인스턴스 번호 부여

데이터 라인의 처음에 '번호:'의 형식으로 해당 인스턴스에 대한 번호를 부여할 수 있다. 인스턴스 번호는 명시된 클래스 내에서 유일한 양수이며, 번호와 콜론(:) 사이에는 공백이 허용되지 않는다. 이와 같이 인스턴스 번호를 부여하는 이유는 추후 객체 참조 관계를 설정하기 위함이다.

    %class employee (name)
    1: 'jordan' 
    2: 'james'  
    3: 'garnett' 
    4: 'malone' 

### 참조 관계 설정

**@** 다음에 참조하는 클래스를 명시하고, 수직바(|) 다음에 참조하는 인스턴스의 번호를 명시하여 객체 참조 관계를 설정할 수 있다. :

    @class_ref | instance_no
    class_ref:
         class_name
         class_id

**@** 다음에는 클래스명 또는 클래스 id를 명시하고, 수직바(|) 다음에는 인스턴스 번호를 명시한다. 수직바(|)의 양쪽에는 공백을 허용하지 않는다.

다음은 *paycheck* 클래스에 인스턴스를 입력하는 예제이며, *name* 속성은 *employee* 클래스의 인스턴스를 참조한다. 마지막 라인과 같이 앞에서 정의되지 아니한 인스턴스 번호를 이용하여 참조 관계를 설정하는 경우 해당 데이터는 **NULL** 로 입력된다. :

    %class paycheck(name department salary)
    @employee|1   'planning'   8000000   
    @employee|2   'planning'   6000000  
    @employee|3   'sales'   5000000  
    @employee|4   'development'   4000000
    @employee|5   'development'   5000000

assign-id-to-class 에서 **%id** 명령어로 *employee* 클래스에 21이라는 식별자를 부여했으므로, 위의 예를 다음과 같이 작성할 수 있다. :

    %class paycheck(name department salary)
    @21|1   'planning'   8000000   
    @21|2   'planning'   6000000  
    @21|3   'sales'   5000000  
    @21|4   'development'   4000000
    @21|5   'development'   5000000

데이터베이스 마이그레이션
=========================

신규 버전의 Rye 데이터베이스를 사용하기 위해서는 기존 버전의 Rye 데이터베이스를 신규 버전의 Rye 데이터베이스로 이전하는 작업을 진행해야 할 경우가 있다. 이때 Rye에서 제공하는 텍스트 파일로 내보내기와 텍스트 파일에서 가져오기 기능을 활용할 수 있다.

**rye unloaddb** 및 **rye loaddb** 유틸리티를 이용하는 마이그레이션 절차를 설명한다.

**권장 시나리오 및 절차**

기존 버전의 Rye가 운영 중인 상태에서 적용할 수 있는 마이그레이션 시나리오를 설명한다. 데이터베이스 마이그레이션을 위해서는 **rye unloaddb**와 **rye loaddb** 유틸리티를 사용한다. 자세한 내용은 unload-db 및 load-db 를 참조한다.

1.  기존 Rye 서비스 종료

    **rye service stop** 을 실행하여 기존 Rye로 운영되는 모든 서비스 프로세스를 종료한 후, Rye 관련 프로세스들이 모두 정상 종료되었는지 확인한다.

    Rye 관련 프로세스들이 모두 정상 종료되었는지 확인하려면, Linux에서는 **ps -ef|grep cub\_**를 실행한다. cub\_로 시작하는 프로세스가 없으면 정상적으로 종료된 것이다. Windows에서는 &lt;Ctrl + Alt + Delete&gt; 키를 누른 후 \[작업 관리자 시작\]을 선택한다. \[프로세스\] 탭에 cub\_로 시작하는 프로세스가 없으면 정상적으로 종료된 것이다. Rye 서비스 종료 후에도 관련 프로세스가 존재하면 Linux에서는 **kill** 명령으로 강제 종료한 후 **ipcs -m** 명령으로 Rye 브로커가 사용 중이던 공유 메모리를 확인하고 삭제한다. Windows에서는 작업 관리자의 \[프로세스\] 탭에서 해당 이미지 이름을 마우스 오른쪽 버튼으로 클릭하고 \[프로세스 끝내기\]를 선택하여 강제 종료한다.

2.  기존 데이터베이스 백업

    **rye backupdb** 유틸리티를 이용하여 기존 버전의 데이터베이스 백업을 수행한다. 그 이유는 데이터베이스 언로드/로드 작업 중 발생 가능한 장애에 대비하기 위함이다. 데이터베이스 백업에 관한 자세한 내용은 db-backup을 참조한다.

3.  기존 데이터베이스 언로드

    **rye unloaddb** 유틸리티를 이용하여 기존 버전의 Rye에서 생성된 데이터베이스를 언로드한다. 데이터베이스 언로드에 관한 자세한 내용은 unload-db 를 참조한다.

4.  기존 Rye의 환경 설정 파일 보관

    **Rye/conf** 디렉터리 아래의 **rye.conf**, **rye\_broker.conf**, **cm.conf** 등의 환경 설정 파일을 보관한다. 이는 기존 Rye 데이터베이스 환경에 적용된 파라미터 설정값을 신규 Rye 데이터베이스 환경에서 편리하게 적용할 수 있기 때문이다.

5.  신규 버전의 Rye 설치

    기존 버전의 Rye에서 생성된 데이터의 백업 및 언로드 작업이 완료되었으므로, 기존 버전의 Rye 및 데이터베이스를 삭제하고 신규 버전의 Rye를 설치한다. Rye 설치에 대한 자세한 내용은 /start을 참조한다.

6.  신규 Rye의 환경 설정

    **기존 Rye의 환경 설정 파일 보관하기** 에서 보관한 기존 데이터베이스의 환경 설정 파일을 참고하여 신규 버전의 Rye 환경을 설정할 수 있다. 환경 설정에 대한 자세한 내용은 "Rye 시작"의 /install을 참조한다.

7.  신규 데이터베이스 로드

    **rye createdb** 유틸리티를 이용하여 데이터베이스를 생성하고, **rye loaddb** 유틸리티를 이용하여 언로드한 데이터를 해당 데이터베이스에 로드한다. 데이터베이스 생성에 대한 자세한 내용은 "관리자 안내서"의 creating-database 을 참조하고, 데이터베이스 로드에 대한 자세한 내용은 load-db를 참조한다.

8.  신규 데이터베이스 백업

    신규 데이터베이스에 데이터 로딩이 완료되면, **rye backupdb** 유틸리티를 이용하여 신규 버전의 Rye 환경에서 생성된 데이터베이스를 백업한다. 그 이유는 기존 버전의 Rye 환경에서 백업한 데이터를 신규 버전의 Rye 환경에서 복구할 수 없기 때문이다. 데이터베이스 백업에 대한 자세한 내용은 db-backup을 참조한다.

같은 버전이라 하더라도 백업 및 복구 시 32비트 데이터베이스 볼륨과 64비트 데이터베이스 볼륨 간에는 상호 호환을 보장하지 않는다. 따라서 32비트 Rye에서 백업한 데이터베이스를 64비트 Rye에서 복구하거나, 이와 반대로 작업하는 것을 권장하지 않는다.


