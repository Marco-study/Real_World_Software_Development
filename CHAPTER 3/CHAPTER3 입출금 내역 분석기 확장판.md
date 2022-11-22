<details>
    <summary> 태현 </summary>

- 태현
    
    ## 목표
    
    코드베이스에 유연성을 추가하고 유지보수성을 개선하는 데 도움을 주는 개방/폐쇄 원칙을 배운다. 또한 언제 인터페이스를 사용해야 좋을지를 설명하는 일반적인 가이드라인과 높은 결합도를 피할 수 있는 기법도 배운다. 
    
    ## 확장된 입출금 내역 분석기 요구 사항
    
    - 특정 입출금 내역을 검색할 수 있는 기능. 예를 들어 주어진 날짜 범위 또는 특정 범주의 입출금 내역 얻기
    - 검색 결과의 요약 통계를 텍스트 HTML 등 다양한 형식으로 만들기
    
    ## 개방/폐쇄 원칙
    
    특정 금액 이상의 모든 입출금 내역을 검색하는 메서드를 구현해보자. 이 메서드는 일종의 처리 기능을 담당하므로 `BankTransactoinProcessor` 클래스 안에 정의하면 나중에 관련 메서드를 조금 더 쉽게 찾을 수 있다. (저 기능 때문에 새로운 클래스를 추가하는 것은 더 복잡해진다.)
    
    **********BankStatementProcessor**********
    
    ```java
    public class BankStatementProcessor {
    
        private final List<BankTransaction> bankTransactions;
    
    		// ... 
    
        public List<BankTransaction> findTransactionsGreaterThanEqual(final int amount) {
            final List<BankTransaction> result = new ArrayList<>();
            for (BankTransaction bankTransaction : bankTransactions) {
                if (bankTransaction.getAmount() >= amount) {
                    result.add(bankTransaction);
                }
            }
            return result;
        }
    
        public List<BankTransaction> findTransactionsInMonth(final Month month) {
            final List<BankTransaction> result = new ArrayList<>();
            for (BankTransaction bankTransaction : bankTransactions) {
                if (bankTransaction.getDate().getMonth() == month) {
                    result.add(bankTransaction);
                }
            }
            return result;
        }
    }
    ```
    
    `findTransactionsGreaterThanEqual()` 메소드는 인자로 들어온 금액과 같거나 큰 금액을 가지는 `BankTransaction`을 리스트에 담아 반환한다.
    
    `findTransactionsInMonth()` 메소드는 인자로 들어온 월과 같은 `BankTransaction`을 리스트에 담아 반환한다.
    
    두 메서드는 `if문` 내부를 제외하고 모두 공통 코드로 이루어져 있다. 
    
    위 두 코드를 다음과 같이 하나의 메서드로 변경할 수 있다.
    
    ```java
    public List<BankTransaction> findTransactionsInMonthAndGreater(final Month month, final int amount) {
        final List<BankTransaction> result = new ArrayList<>();
        for (BankTransaction bankTransaction : bankTransactions) {
            if (bankTransaction.getAmount() == amount && bankTransaction.getDate().getMonth() == month) {
                result.add(bankTransaction);
            }
        }
        return result;
    }
    ```
    
    하지만 위 코드는 문제점이 있다. 
    
    - 거래 내역의 여러 속성을 조합할수록 코드가 점점 복잡해진다.
    - 반복 로직과 비즈니스 로직이 결합되어 분리하기가 어려워진다.
    - 코드를 반복한다.
    
    개방/폐쇄 원칙은 이런 상황에 적용된다. 개방/폐쇄 원칙을 적용하면 코드를 직접 바꾸지 않고 해당 메서드나 클래스의 동작을 바꿀 수 있다. 
    
    **BankTransactionFilter**
    
    ```java
    @FunctionalInterface
    public interface BankTransactionFilter {
        boolean test(BankTransaction bankTransaction);
    }
    ```
    
    위와 같이 함수형 인터페이스를 만든다. (`Predicate`로 대체 가능하다.)
    
    그리고 `BankTransactionFilter`를 인자로 받는 메서드를 구현한다.
    
    ```java
    public List<BankTransaction> findTransactions(final BankTransactionFilter bankTransactionFilter) {
        final List<BankTransaction> result = new ArrayList<>();
        for (BankTransaction bankTransaction : bankTransactions) {
            if (bankTransactionFilter.test(bankTransaction)) {
                result.add(bankTransaction);
            }
        }
        return result;
    }
    ```
    
    위처럼 코드를 작성하면 `BankTransaction`을 필터링 하는 조건이 추가된다 하여도 위 메서드를 수정할 필요가 없이 인자로 들어오는 값만 변경하면 된다. 그리고 비즈니스 로직을 담당하는 `BankTransactionFilter`와 반복하는 로직을 분리한다. → 변경 없이 확장된다.
    
    ### 함수형 인터페이스 인스턴스 만들기
    
    ```java
    public class BankTransactionIsInFebruaryAndExpensive implements BankTransactionFilter {
    
        @Override
        public boolean test(BankTransaction bankTransaction) {
            return bankTransaction.getDate().getMonth() == Month.FEBRUARY
                    && bankTransaction.getAmount() >= 1000;
        }
    }
    ```
    
    위 코드처럼 함수형 인터페이스를 구현하여 `findTransactions()` 메서드의 인수로 전달하여 자유롭게 사용할 수 있다.
    
    ```java
    final List<BankTransaction> transactions = bankStatementProcessor.findTransactions(
     new bankTransactionIsInFebruaryAndExpensive());
    ```
    
    ### 람다 표현식
    
    자바 8부터는 람다 표현식을 사용하여 별도의 클래스 생성 없이 함수형 인터페이스를 구현할 수 있다.
    
    ```java
    final List<BankTransaction> transactions = bankStatementProcessor.findTnrasactions(
    bankTransaction -> bankTransaction.getDate().getMonth() == Month.FEBRUARY);
    ```
    
    요악하면 다음과 같은 장점 덕분에 개방/폐쇄 원칙을 사용한다.
    
    - 기존 코드를 바꾸지 않으므로 기존 코드가 잘못될 가능성이 줄어든다.
    - 코드가 중복되지 않으므로 기존 코드의 재사용성이 높아진다. → 반복하는 로직을 재사용
    - 결합도가 낮아지므로 코드 유지보수성이 좋아진다. → 반복하는 로직과 비즈니스 로직의 결합도가 낮아짐
    
    ## 인터페이스 문제
    
    ### 문제점
    
    현재 기능 확장에 중점을 둔 메서드를 만듦으로써 새로운 문제가 생겼다. 기존에 작성한 세 메서드에 대한 처리때문이다. 한 인터페이스에 모든 기능을 추가하는 갓 인터페이스를 만드는 일은 피해야 한다.
    
    ### 갓 인터페이스
    
    `BankTransactionProcessor`에서 제공하는 모든 API를 한 인터페이스에 모으게 되면 인터페이스가 복잡해진다. 
    
    뿐만 아니라 다음과 같은 문제도 존재한다.
    
    - 구현 클래스는 인터페이스에 정의한 모든 메서드를 구현해야 하는데 인터페이스가 바뀌면 이를 구현한 코드도 바뀐 내용을 지원하도록 갱신해야 한다. → 문제 발생 가능성 UP
    - 월, 카테고리 같은 `BankTransaction`의 속성이 `calculateAverageForCategory()`, `calculateTotalInMonth`  처럼 메서드 이름의 일부로 사용되었다. 인터페이스가 도메인 객체의 특정 접근자에게 종속되는 문제가 생겼다.
    
    이런 이유에서 보통 작은 인터페이스를 권장한다. 그래야 도메인 객체의 다양한 내부 연산으로의 디펜던시를 최소화할 수 있다.
    
    ### 지나친 세밀함
    
    다음 인터페이스처럼 지나치게 작은 인터페이스를 만든다면 어떨까?
    
    ```java
    interface CalculateTotalAmount {
    	double calculateTotalAmount();
    }
    
    interface CalculateAverage{
    	double calculateAverage();
    }
    
    interface CalculateTotalInMonth {
    	double calculateTotalInMonth();
    }
    ```
    
    지나치게 세밀한 인터페이스도 유지보수에 방해가 된다. 실제로 위 코드는 안티 응집도 문제가 발생한다. 즉, 기능이 여러 인터페이스로 분산되므로 필요한 기능을 찾기가 어렵다. 
    
    ## 명시적 API vs 암묵적 API
    
    일반적인 `findTransactions()` 메서드를 쉽게 정의할 수 있는 상황에서 `findTransactionsGreaterThanEqual()`처럼 구체적으로 메서드를 정의해야 하는지 의문이 생긴다. 이런 딜레마를 명시적 API 제공 vs 암묵적 API 제공 문제라고 부른다. 
    
    명시적 API를 사용한다면 메서드가 어떤 동작을 하는지 잘 설명되어 있고 사용하기 쉽다. API의 가독성을 높이고 쉽게 이해하도록 메서드 이름을 서술적으로 만들었다. 하지만 이 메서드의 용도가 특정 상황에 국한되어 각 상황에 맞는 새로운 메서드를 많이 만들어야 하는 상황이 벌어진다. 
    
    반면, 암묵적 API는 처음에 사용하기가 어렵고, 문서화를 잘 해놓아야 한다. 하지만 거래 내역을 검색하는 데 필요한 모든 상황을 단순한 API로 처리할 수 있다. 
    
    → 상황에 따라 어떤 것이 더 좋을지 바뀔 수 있다.
    
    ```java
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  public class BankStatementProcessor {
    
        private final List<BankTransaction> bankTransactions;
    
        public BankStatementProcessor(final List<BankTransaction> bankTransactions) {
            this.bankTransactions = bankTransactions;
        }
    
        public double summarizeTransactions(final BankTransactionSummarizer bankTransactionSummarizer) {
            double result = 0;
            for (final BankTransaction bankTransaction : bankTransactions) {
                result = bankTransactionSummarizer.summarize(result, bankTransaction);
            }
            return result;
        }
    
        public double calculateTotalInMonth(final Month month) {
            return summarizeTransactions((result, transaction) ->
                    transaction.getDate().getMonth() == month ? result + transaction.getAmount() : result);
        }
    
        public double calculateTotalForCategory(final String category) {
            double total = 0;
            for (final BankTransaction bankTransaction : bankTransactions) {
                if (bankTransaction.getDescription().equals(category)) {
                    total += bankTransaction.getAmount();
                }
            }
            return total;
        }
    
        public List<BankTransaction> findTransactions(final BankTransactionFilter bankTransactionFilter) {
            final List<BankTransaction> result = new ArrayList<>();
            for (BankTransaction bankTransaction : bankTransactions) {
                if (bankTransactionFilter.test(bankTransaction)) {
                    result.add(bankTransaction);
                }
            }
            return result;
        }
    
        public List<BankTransaction> findTransactionsGreaterThanEqual(final int amount) {
            return findTransactions((transaction) -> transaction.getAmount() >= amount);
        }
    
    }
    ```
    
    ### 도메인 클래스 vs 원싯값
    
    ```java
    @FunctionalInterface
    public interface BankTransactionSummarizer {
        double summarize(double accumulator, BankTransaction bankTransaction);
    }
    ```
    
    위 코드처럼 `double` 이라는 원시값을 반환하는 것은 일반적으로 좋은 방법이 아니다. 원시값은 다양한 결과를 반환할 수 없어 유연성이 떨어지기 때문이다. 
    
    `double`을 감싸는 도메인 클래스 `Summary`를 만들어 이 문제를 해결할 수 있다. 새 클래스에 필요한 필드와 결과를 언제든지 추가할 수 있고 이 기법을 이용하면 도메인의 다양한 개념간의 결합을 줄이고, 요구 사항이 바뀔 때 연쇄적으로 코드가 바뀌는 일도 최소화할 수 있다. 
    
    ## 다양한 형식으로 내보내기
    
    선택된 입출금 목록의 요약 통계를 텍스트, HTML, JSON 등 다양한 형식으로 내보내야 한다.
    
    ### 도메인 객체 소개
    
    숫자, 컬렉션, 특별한 도메인 객체, 더 복잡한 도메인 객체 등으로 입출금 목룍의 요약 통계를 내보낼 형식을 지정할 수 있다.
    
    ************************************************특별한 도메인 객체************************************************
    
    ```java
    public class SummaryStatistics {                                                          
        public double getMin() {
            return min;
        }
    
        public double getAverage() {
            return average;
        }
    }
    ```
    
    ### 적절하게 인터페이스를 정의하고 구현하기
    
    `Export`라는 인터페이스를 정의해 다양한 내보내기 구현 코드가 다른 코드와 결합하지 않도록 방지한다. JSON으로 내보내든, XML로 내보내든 같은 인터페이스를 구현하면 되므로 다양한 내보내기 기능을 편리하게 구현할 수 있다. 
    
    ****************Exporter****************
    
    ```java
    public interface Exporter {
    		void export(SummaryStatistics summaryStatistics);
    }
    ```
    
    위 코드처럼 구현하면 안된다. 
    
    - `void` 반환 형식은 아무 도움이 되지 않고, 기능을 파악하기도 어렵다.
    - `void`를 반환하면 결과를 테스트하기도 어렵다.
    
    ****************Exporter****************
    
    ```java
    public interface Exporter {
    		String export(SummaryStatistics summaryStatistics);
    }
    ```
    
    따라서 위처럼 반환 타입을 지정하는 것이 좋다. 
    
    ## 예외 처리
    
    프로그램을 실행할 때 발생할 수 있는 다양한 예외를 처리할 수 있어야 한다.
    
    ### 예외를 사용해야 하는 이유
    
    CSV 파일을 파싱하는 `BankStatementCSVParser`를 살펴보자. 다음과 같은 문제가 발생할 수 있다.
    
    - CSV 행의 열이 세 개를 초과한다.
    - CSV 행의 열이 세 개 미만이다.
    - 날짜 등 일부 열의 자료 형식이 잘못되었다.
    
    자바는 예외를 일급 언어 기능으로 추가하고 다음과 같은 장점을 제공한다.
    
    - **문서화 :** 메서드 시그니처 자체에 예외를 지원한다.
    - **형식 안전성 :** 개발자가 예외 흐름을 처리하고 있는지를 형식 시스템이 파악한다.
    - **관심사 분리 :** 비즈니스 로직과 예외 회복이 각각 `try/catch` 블록으로 구분된다.
    
    자바는 두 가지 종료의 예외를 지원한다.
    
    - **************************************확인된 예외(Checked Exception) :************************************** 회복해야 하는 대상의 예외다. 자바에서는 메서드가 던질 수 있는 확인된 예외 목록을 선언해야 한다. 아니면 해당 예외를 `try/catch`로 처리해야 한다.
        - `Exception`
    - **********************************미확인 예외(Unchecked Exception) :********************************** 프로그램을 실행하면서 언제든 발생할 수 있는 종류의 예외다. 확인된 예외와 달리 메서드 시그니처에 명시적으로 오류를 선언하지 않으면 호출자도 이를 꼭 처리할 필요가 없다.
        - `Error`, `RuntimeException`
    
    ### 미확인 예외와 확인된 예외에서 선택하기
    
    일시적으로 발생하는 오류라면 동작을 다시 시도하거나 화면에 메시지를 출력해 응용프로그램의 반응성을 훌륭하게 유지할 수 있다. 보통 비즈니스 로직 검증 시 발생한 문제는 불필요한 `try/catch` 구문을 줄일 수 있도록 미확인 예외로 결정한다. 또한 예외가 발생했을 때 응용프로그램을 어떻게 회복시킬 것인지 애매한 상황도 있다. 이런 상황에서는 API 사용자에게 오류를 복구하라고 강제할 필요가 없다. 게다가 시스템 오류(저장공간이 꽉 참)가 발생했을 때 사용자가 할 수 있는 일이 없으므로 시스템 오류도 미확인 예외로 지정한다. 즉, 대다수의 예외를 미확인 예외로 지정하고 꼭 필요한 상황에서만 확인된 예외로 지정해 불필요한 코드를 줄여야 한다. 
    
    **************************************과도하게 세밀함**************************************
    
    검증 로직은 객체를 생성하는 곳에 추가할 수 있지만 전용 Validator를 만드는 것을 추천한다. 
    
    - 검증 로직을 재사용해 코드를 중복하지 않는다.
    - 시스템의 다른 부분도 같은 방법으로 검증할 수 있다.
    - 로직을 독립적으로 유닛 테스트할 수 있다.
    - 이 기법은 프로그램 유지보수와 이해하기 쉬운 SRP를 따른다.
    
    다음 예외 처리를 보자.
    
    ```java
    public class OverlySpecificBankStatementValidator {
    
    		private String description;
    		private String date;
    		private String amount;
    
    		// 생성자
    
    		public boolean validate() throws DescriptionToooLongException, InvalidDateFormat,
    																		 DateInTheFutureException, InvalidAmountException {
    				if (this.descript.length() > 100) {
    						throw new DescriptionTooLongException();
    				}
    			
    				final LocalDate parsedDate;
    				try {
    						parseDate = LocalDate.parse(this.date);
    				}
    				catch (DateTimeParseException e) {
    						throw new InvalidDateFormat();
    				}
    				if (parsedDate.isAfter(LocalDate.now())) throw new DateInTheFutureException();
    				
    				try {
    						Double.parseDouble(this.amount);
    				}
    				catch (NumberFormatException e) {
    						throw new InvalidAmountException();
    				}
    				return true;
    		}
    }
    ```
    
    위 `OverlySpecificBankStatementValidator`에서 던지는 4개의 예외는 모두 `Exception`을 상속받는 `CheckedException`이다. 위 방식대로 구현하면 각각의 예외에 적합하고 정확한 회복 기법을 구현할 수 있지만 너무 많은 설정 작업이 필요하고, 여러 예외를 선언해야 하며, 사용자가 이 모든 예외를 처리해야 하므로 생산성이 현저하게 떨어진다. 
    
    ************************************과도하게 덤덤함************************************
    
    모든 예외를 `IllegalArgumentException` 등의 미확인 예외로 지정하는 극단적인 상황도 있다. 
    
    ```java
    public class OverlySpecificBankStatementValidator {
    
    		private String description;
    		private String date;
    		private String amount;
    
    		// 생성자
    
    		public boolean validate() throws DescriptionToooLongException, InvalidDateFormat,
    																		 DateInTheFutureException, InvalidAmountException {
    				if (this.descript.length() > 100) {
    						throw new IllegalArgumentException("...");
    				}
    			
    				final LocalDate parsedDate;
    				try {
    						parseDate = LocalDate.parse(this.date);
    				}
    				catch (DateTimeParseException e) {
    						throw new IllegalArgumentException("...");
    				}
    				if (parsedDate.isAfter(LocalDate.now())) throw new DateInTheFutureException();
    				
    				try {
    						Double.parseDouble(this.amount);
    				}
    				catch (NumberFormatException e) {
    						throw new IllegalArgumentException("...");
    				}
    				return true;
    		}
    }
    ```
    
    **노티피케이션 패턴**
    
    노티피케이션 패턴은 너무 많은 미확인 예외를 사용하는 상황에 적합한 해결책을 제공한다. 이 패턴에서는 도메인 클래스로 오류를 수집한다. 
    
    ```java
    public class Notification {
    		private final List<String> errors = new ArrayList<>();
    
    		public void addError(final String message) {
    				errors.add(message);
    		}
    
    		public boolean hasErrors() {
    				return !errors.isEmpty();
    		}
    
    		public String errorMessage() {
    				return errors.toString();
    		}
    	
    		public List<String> getErrors() {
    				return this.errors;
    		}
    }
    ```
    
    이 클래스를 정의해 한 번에 여러 오류를 수집할 수 있는 검증자를 만든다. 이제 예외를 던지지 않고 `Notification` 객체에 메시지를 추가한다. 
    
    ```java
    public Notification validate() {
    
    		final Notification notification = new Notification();
    		if (this.description.length() > 100) {
    				notification.addError("The description is too long");
    		}
    
    		final LocalDate parsedDate;
    		try {
    				parsedDate = LocalDate.parse(this.date);
    				if (parsedDate.isAfter(LocalDate.now())) {
    						notification.addError("date cannot be in the future");
    				}
    		}
    		catch (DateTimeParseException e) {
    				notification.addError("Invalid format for date");
    		}
    		final double amount;
    		try {
    				amount = Double.parseDouble(this.amount);
    		}
    		catch (NumberFormatException e) {
    				notification.addError("Invalid format for amount");
    		}
    		return notification;
    }
    ```
    
    ### 예외 사용 가이드라인
    
    ******************************************************예외를 무시하지 않음******************************************************
    
    문제의 근본 원인을 알 수 없다고 예외를 무시하면 안된다. 예외를 처리할 수 있는 방법이 명확하지 않으면 미확인(Unchecked Exception) 예외를 대신 던진다. 
    
    **********************************************************************일반적인 예외는 잡지 않음**********************************************************************
    
    가능한 구체적으로 예외를 잡으면 가독성이 높아지고 더 세밀하게 예외를 처리할 수 있다. 
    
    ********************예외 문서화********************
    
    API 수준에서 미확인 예외를 포함한 예외를 문서화하므로 API 사용자에게 문제 해결의 실마리를 제공한다.
    
    ```java
    @throws NoSuchFileException
    @throws DirectoryNotEmptyExceptin 
    ...
    ```
    
    ****************************************************************************************************특정 구현에 종속된 예외를 주의할 것****************************************************************************************************
    
    특정 구현에 종속된 예외를 던지면 캡슐화가 깨지므로 주의하자.
    
    ```java
    public String read(Source source) throws OracleException {...}
    ```
    
    **************************************예외 vs 제어 흐름**************************************
    
    예외로 흐름을 제어하지 않는다. 
    
    ```java
    try {
    		while (true) {
    				System.out.println(source.read());
    		}
    }
    catch (NoDataException e) {}
    ```
    
    이런 종류의 코드는 다음과 같은 여러 문제를 일으키므로 피해야 한다. 
    
    - 예외를 처리하느라 불필요한 `try/catch` 문법이 추가되어 코드 가독성을 떨어뜨린다.
    - 코드의 의도를 이해하기 어려워진다.
    - 예외가 발생했을 때 스택 트레이스 생성, 보존과 관련된 부담이 생긴다.
    
    ### 예외 대안 기능
    
    예외를 대체할 만한 다양한 기능에 대해 알아보자
    
    **********************null 사용**********************
    
    ```java
    final String[] columns = line.split(",");
    
    if (cloumns.length < EXPECTED_ATTRIBUTES_LENGTH) {
    		return null;
    }
    ```
    
    이 방법은 절대 사용하지 말아야 한다. `null`을 사용하면 의도를 파악하기 어렵고 `null` 체크를 하지 않으면 오류가 발생할 수 있다. 
    
    **********************************null 객체 패턴**********************************
    
    객체가 존재하지 않을 때 `null`을 반환하는 대신에 필요한 인터페이스를 구현하는 객체(바디는 비어 있음)를 반환하는 기법이다. 이 기법을 사용하면 `NullPointerException`을 피할 수 있고 `null` 확인 코드를 제거할 수 있다. 단점은 데이터에 문제가 있어도 빈 객체를 이용해 실제 문제를 무시할 수 있다.
    
    **********************Optional<T>**********************
    
    자바 8에서는 값이 없는 상태를 표현하는 내장 데이터 형식인 `Optional<T>`을 선보였다. 
    
    ********Try<T>********
    
    성공하거나 실패할 수 있는 연산을 가리키는 `Try<T>` 데이터 형식도 있다. `Optional`과 비슷하지만 값이 아니라 연산에 적용한다는 점이 다르다. JDK는 지원하지 않으므로 외부 라이브러리를 이용해야 한다.
    
    ## 빌드 도구 사용
    
    ### 왜 빌드 도구를 사용할까?
    
    응용 프로그램을 실행하려면 코드를 구현하고 컴파일 해야한다. 이 때 컴파일 명령에 대해 숙지해야 하며 다른 라이브러리를 사용한다면 디펜던시를 스스로 관리해야 한다. 또한 프로젝트를 WAR나 JAR 형식으로 패키징해야 하는 등 해결해야 할 문제가많다. 
    
    빌드 도구룰 사용하면 응용프로그램 빌드, 테스트, 배포 등 소프트웨어 개발 생명 주기를 자동화할 수 있도록 도와준다. 
    
    **빌드 도구의 장점**
    
    - 프로젝트에 적용되는 공통적인 구조를 제공하기 때문에 여러 개발자가 프로젝트를 더 편안하게 받아들일 수 있다.
    - 응용프로그램을 빌드하고 실행하는 반복적이고, 표준적인 작업을 설정한다.
    - 저수준 설정과 초기화에 들이는 시간을 절약하므로 개발에만 집중할 수 있다.
    - 잘못된 설정이나 일부 빌드 과정 생략 등으로 발생하는 오류의 범위를 줄인다.
    - 공통 빌드 작업을 재사용해 이를 다시 구현할 필요가 없으므로 시간을 절약한다.
    
    ### 메이븐 사용
    
    XML 기반으로 빌드 과정을 정의하는 빌드 툴이다.  
    
    ****************************프토젝트 구조****************************
    
    - src/main/java : 프로젝트에 필요한 모든 자바 클래스를 개발해 저장하는 폴더
    - src/test/java : 프로젝트의 테스트 코드를 개발해 저장하는 폴더
    - src/main/resources : 응용프로그램에서 사용하는 텍스트 파일 등 추가 자원을 포함하는 폴더
    - src/test/resources : 테스트에서 사용할 추가 자원을 포함하는 폴더
    
    pom.xml 파일을 만들어 응용프로그램 빌드에 필요한 과정을 다양한 XML 정의로 지정해 빌드 프로세스를 정의한다. 
    
    **************pom.xml**************
    
    빌드 과정을 지정하는 XML 파일이다.
    
    ```xml
    <project xmlns=...>
    <groupId>com...</groupId>
    <artifactId>...</artifactId>
    <version>...</version>
    <build>
    	<plugins>
    		<plugin>
    		...
    		</plugin>
    	</plugins>
    </build>
    
    <dependencies>
    	<dependency>
    		...
    	</dependency>
    </depenencies>
    ```
    
    - project
        - pom.xml 파일의 최상위 수준 요소
    - groupId
        - 프로젝트를 만드는 조직의 고유 식별자
    - artifactId
        - 빌드 과정에서 생성된 부산물의 고유한 기본 이름
    - packaging
        - 부산물에 사용할 패키지 형식(Jar,War)을 지정
        - default : Jar
    - version
        - 프로젝트에서 생성하는 부산물 버전
    - bulid
        - 플러그인, 지원 등 빌드 과정을 가이드하는 설정
    - dependencies
        - 프로젝트 디펜던시 목록
    
    ******************************메이븐 명령어******************************
    
    - `mvn clean`: 빌드하기 전에 기존 빌드에서 생성된 부산물을 초기화
    - `mvn compile` :  프로젝트의 소스코드를 컴파일(target 폴더에)
    - `mvn test` : 컴파일된 소스코드를 테스트
    - `mvn package` : JAR와 같은 적절한 형식으로 컴파일된 코드를 패키징
    
    ### 그레이들 사용
    
    Groovy, Kotlin 언어 등을 이용해 DSL을 적용한다. 메이븐보다 더 자연스럽게 빌드를 지정하고, 쉽게 커스터마이즈할 수 있으며 쉽게 이해할 수 있다. 게다가 캐시, 점진적 컴파일 등 빌드 시간을 단축하는 기능도 지원한다.
    
    ************************build.gradle************************
    
    pom.xml과 같은 역할을 한다. 그레이들에서는 build.gradle 파일에 빌드 과정을 지정한다. 또한 여러 프로젝트 빌드와 설정 변수를 포함하는 settings.gradle 파일도 있다. 
    
    ```groovy
    apply plugin: 'java'
    apply plugin: 'application'
    
    group = 'com..'
    version = '...'
    
    sourceCompatibility = ...
    targetCompatibility = ...
    
    mainClassName = "..."
    
    repositories {
    	mavenCentral()
    }
    
    dependencies {
    	testImplementation group : ...
    }
    ```
    
    ************명령어************
    
    - `gradle clean` : 이전 빌드에서 생성된 파일 정리
    - `gradle build` : 응용프로그램을 패키징
    - `gradle test` : 테스트를 실행
    - `gradle run` : application 플러그인의 mainClassName으로 지정된 메인 클래스를 실행



</details>


<details>
    <summary> 성준 </summary>

- 성준
    
    # 입출금 내역 확장판
    
    ## 이번 장의 주요 내용
    
    1. 요구 사항
    2. 개방/폐쇄 원칙
        1. 함수형 인터페이스
        2. 람다 표현식
    3. 도메인 클래스 vs 원싯값
    4. 예외 처리
    
    ## 1. 요구 사항
    
    1. 현재는 특정 월이나 범주로만 딱 조회 가능 → 동적으로 날짜 범위나 범주를 넣고 싶음
    2. 결과를 다양한 형식으로 반환 가능하게 할 것
    
    ## 2. 개방/폐쇄 원칙
    
    ```java
    // 특정 범주 이상의 합계 검색
    public List<BankTransaction> findTransactionsGreaterThanEqual(final int amount) {
    		final List<BankTransaction> result = new ArrayList<>();
    		for (final BankTransaction bankTransaction : bankTransactions) {
    			if (bankTransaction.getAmount() >= amount) {
    				result.add(bankTransaction);
    			}
    		}
    		return result;
    	}
    
    // 특정 월의 합계 검색
    public List<BankTransaction> findTransactionsInMonth (final Month month) {
    		final List<BankTransaction> result = new ArrayList<>();
    		for (final BankTransaction bankTransaction : bankTransactions) {
    			if (bankTransaction.getDate().getMonth() == month) {
    				result.add(bankTransaction);
    			}
    		}
    		return result;
    	}
    ```
    
    → 비슷한 코드가 계속 반복된다. 만약 특정 달 + 특정 범주 이상과 같은 요구사항이 새로 생기면 또 비슷한 코드를 만들어줘야한다. 추상화가 되어 있지 않기 때문이다.
    
    → 현재 코드에서 비즈니스 로직은 “입출금 내역이 특정 달이 맞으면”, “입출금 내역이 특정 범주 이상이 맞으면” 을 판별해주는 로직이다. 이 로직들을 추상화해보자.
    
    ### 2.1 함수형 인터페이스
    
    ```java
    @FunctionalInterface
    public interface BankTransactionFilter {
       boolean test(BankTransaction bankTransaction);
    }
    ```
    
    → 이전에는 BankTransaction 의 특정 필드에 종속적인 로직이었다면 함수형 인터페이스를 쓰면 그렇지 않다. (제너릭 이용하여 파라미터의 타입에도 종속적이지 않을 수 있음.)
    
    → 특정 필드에 종속적이지 않기 때문에 요구사항의 변화가 있어도 이 코드는 변하지 않는다.
    
    ```java
    public List<BankTransaction> findTransactions(final BankTransactionFilter bankTransactionFilter) {
       final List<BankTransaction> result = new ArrayList<>();
       for (final BankTransaction bankTransaction : bankTransactions) {
          if (bankTransactionFilter.test(bankTransaction)) {
             result.add(bankTransaction);
          }
       }
       return result;
    }
    ```
    
    → 기존에는 반복 로직과 판별 로직이 결합되어 있어서 둘 중 하나만 수정하고 싶은 내용이 생겨도 함께 수정사항이 생겼다. 변경 후에는 반복 로직과 판별 로직이 분리되어 그렇지 않다.
    
    ### 2.2 람다 표현식
    
    위에서 구현한 BankTransactionFilter 을 사용하려면 원하는 조건일 때 참 값을 반환하도록 구현해야한다. 요구사항이 생길 때마다 매번 클래스를 계속 생성하면서 구현하는 건 귀찮고 관리해야할 클래스가 늘어나는 귀찮은 일이다.
    
    람다 표현식을 이용하자.
    
    ## 3. 도메인 클래스 vs 원싯값
    
    BankTransactionProcessor 의 최종 코드를 한번 보자.
    
    ```java
    @FunctionalInterface
    public interface BankTransactionSummarizer {
       double summarize(double accumulator, BankTransaction bankTransaction);
    }
    
    @FunctionalInterface
    public interface BankTransactionFilter {
    	boolean test(BankTransaction bankTransaction);
    }
    
    public class BankTransactionProcessor {
    
    	private final List<BankTransaction> bankTransactions;
    
    	public BankTransactionProcessor(final List<BankTransaction> bankTransactions){
    		this.bankTransactions = bankTransactions;
    	}
    
    	public double summarizeTransactions(final BankTransactionSummarizer bankTransactionSummarizer) {
    		double result = 0;
    		for (final BankTransaction bankTransaction : bankTransactions) {
    			result = bankTransactionSummarizer.summarize(result, bankTransaction);
    		}
    		return result;
    	}
    
    	public double calculateTotalInMonth(final Month month) {
    		return summarizeTransactions((acc, bankTransaction) ->
    			bankTransaction.getDate().getMonth() == month ? acc + bankTransaction.getAmount() : acc);
    	}
    
    	public List<BankTransaction> findTransactions(final BankTransactionFilter bankTransactionFilter) {
    		final List<BankTransaction> result = new ArrayList<>();
    		for (final BankTransaction bankTransaction : bankTransactions) {
    			if (bankTransactionFilter.test(bankTransaction)) {
    				result.add(bankTransaction);
    			}
    		}
    		return result;
    	}
    
    	public List<BankTransaction> findTransactionsGreaterThanEqual(final int amount) {
    		return findTransactions(bankTransaction -> bankTransaction.getAmount() >= amount);
    	}
    }
    ```
    
    → summarizeTransactions 메서드와 BankTransactionSummarizer 함수형 인터페이스의 사용으로 반복과 합 로직의 분리가 잘 이루어짐을 알 수 있다.
    
    → BankTransactionSummarizer 의 반환값이 double 로 종속되어 있는 점이 OCP 를 위배한다.
    
    추가적인 요구사항이 생겨 다양한 결과를 포함해야한다면, BankTransactionSummarizer 를 사용하는 모든 코드들을 수정해줘야한다. double 을 감싸는 Summary 객체를 만들어서 관리하자. 새로운 요구사항이 생기면 Summary  만 변경하면 된다.


</details>

<details>
    <summary> 수영 </summary>

- 수영
    # 3장 입출금 내역 확장판

    <aside>
    💡 이전 장에서 **SRP**를 이용한 유지보수 개선과 피해야할 **안티 패턴 (갓 클래스, DRY 원칙)** 등에 대해서 배웠다. → **그럼에도 현재 기능은 상당히 제한적.**

    **이번 장에서 배울것**
    - **OCP**를 이용한 유연성 추가와 유지보수 개선
    - **언제 인터페이스를 사용해야 좋을지**
    - **높은 결합도**를 피할 수 있는 기법
    - 언제 API에 예외를 포함하거나 포함하지 않을지를 결정하는 **예외 처리 방법**
    - Maven, Gradle 같은 검증된 빌드 툴을 이용한 시스템적 **자바 프로젝트 빌드 방법**

    </aside>

    # 확장된 입출금 내역 분석기 요구 사항

    1. 특정 입출금 내역을 검색할 수 있는 기능. 예를 들어 주어진 날짜 범위 또는 특정 범주의 입출금 내역 얻기.
        
        → 너무 경우가 많아서 코드의 반복으로 짜려면 정말 무수히 많은 코드를 짜야한다.
        
        → 외부에서 조건을 넣을 수 있게 Predicate같은 함수형 인터페이스를 사용하는 것이 좋을 것.
        
    2. 검색 결과의 요약 통계를 텍스트, HTML 등 다양한 형식으로 만들기.
        
        → 다양한 형식을 출력하는 부분이 필요할 듯 (인터페이스??)
        

    ```java
    // 특정 금액 이상의 은행 거래 내역 찾기
        public List<BankTransaction> findTransactionsGreaterThanEqual(final int amount) {
            final List<BankTransaction> result = new ArrayList<>();
            for(final BankTransaction bankTransaction : bankTransactionList) {
                if (bankTransaction.getAmount()>= amount) {
                    result.add(bankTransaction);
                }
            }

            return result;
        }
        
        // 특정 달의 입출금 내역 찾기
        public List<BankTransaction> findTransactionsInMonth(final Month month) {
            final List<BankTransaction> result = new ArrayList<>();
            for(final BankTransaction bankTransaction : bankTransactionList) {
                if (bankTransaction.getDate().getMonth() == month) {
                    result.add(bankTransaction);
                }
            }
            
            return result;
        }
    ```

    중복되는 코드가 너무 많다. ⇒ 소프트웨어가 불안정해짐. ( 요구사항 수정 or 추가 시 영향 증가)

    왜? → 그냥 반복적으로 순회하던걸 짝수 번째만 계산한다하면 모든 반복문을 수정해야함.

    어떻게 바꿀 수 있을까? OR이나 AND 막 붙이면서 분기문을 계속 수정한다??

    완전 나이브하게 생각하면 이렇게 생각할 수 있지만 이것도 이전 코드랑 다를 바가 없음.

    ## 이전 방법들의 한계

    - 거래 내역의 여러 속성을 조합 → 코드가 점점 복잡해짐.
    - 반복 로직과 비즈니스 로직의 결합으로 분리가 어려워짐.
    - 코드 반복

    ⇒ 이걸 해결할 수 있는게 **OCP**

    ⇒ 코드 복사나 새 파라미터 추가 등의 코드를 바꾸지 않고 확장 가능

    자, 그럼 OCP로 뭐를 바꿔야 코드 수정없이 확장이 가능할까? 위에 한계들일 것이다.

    그 중에 **반복 로직이랑 비즈니스 로직을 분리** 시키면 해결가능할 것 같음. → **인터페이스**

    # 인터페이스를 통한 OCP를 적용해 개선해보기

    ```java
    public class BankStatementProcessor {

        private final List<BankTransaction> bankTransactionList;

        public BankStatementProcessor(List<BankTransaction> bankTransactionList) {
            this.bankTransactionList = bankTransactionList;
        }

        public List<BankTransaction> findTransactions(BankTransactionFilter bankTransactionFilter) {
            List<BankTransaction> result = new ArrayList<>();
            for (BankTransaction bankTransaction : bankTransactionList) {
                if (bankTransactionFilter.test(bankTransaction)) {
                    result.add(bankTransaction);
                }
            }
            return result;
        }
    }

    ---

    @FunctionalInterface // 어노테이션으로 명확하게 표현
    public interface BankTransactionFilter {
        boolean test(BankTransaction bankTransaction);
    }
    ```

    저렇게 Filter를 만들고 구현체를 계속 만들어 가면 다 좋은데 클래스가 너무 많아져서 코드가 애플리케이션이 되게 복잡해지고 클래스를 반복해서 만드는 것 또한 귀찮다.

    # 람다 표현식으로 간단하게 표현해보기

    ```java
    List<BankTransaction> transactions = bankStatementProcessor.findTransactions(bankTransaction -> 
    bankTransaction.getDate().getMonth() == Month.NOVEMBER && bankTransaction.getAmount() >= 1000
    );
    ```

    새로운 조건이 생길때마다 구현체를 만들어서 test를 구현하는 것이 아니라 입력 시에 조건을 바로 넣는다. 람다 표현식을 이용하면 부가적으로 귀찮게 클래스를 만들 필요가 없어진다.

    > 이전까지는 OCP를 통한 유연성 추가와 유지 보수 개선을 알아본것이다. 비즈니스 로직과 반복문을 분리한 것이다.
    1. 인터페이스를 통해 OCP를 성립시켰다. → 구현체 클래스 만들기가 귀찮음.
    2. 람다 표현식을 통해 구현체 클래스를 만들지 않게 만들었다.
    > 

    이제는 저번 장에서 만들었던 계산 로직을 바꿔보자.

    ```java
            public double calculateTotalAmount() {
            double total = 0;
            for (final BankTransaction bankTransaction : bankTransactionList) {
                total += bankTransaction.getAmount();
            }

            return total;
        }

        public double calculateTotalInMonth(final Month month) {
            double total = 0;
            for (final BankTransaction bankTransaction : bankTransactionList) {
                if (bankTransaction.getDate().getMonth() == month) {
                    total += bankTransaction.getAmount();
                }
            }
    
            return total;
        }

        public double calculateTotalForCategory(final String category) {
            double total = 0;
            for (final BankTransaction bankTransaction : bankTransactionList) {
                if (bankTransaction.getDescription().equals(category)) {
                    total += bankTransaction.getAmount();
                }
            }
            return total;
        }
    ```

    ⇒ 만약 똑같이 더 많은 조건이 생기면 엄청 커질 것이다. → 결국 갓 인터페이스가 된다.

    - 인터페이스가 바뀌면 구현 클래스가 바껴야 해서 연산이 많이 추가될수록 코드가 자주 바뀌게 된다. 즉, 문제가 발생할 수 있는 범위가 넓어진다.
    - 속성이 메서드 이름의 일부로 사용 → 인터페이스가 도메인 객체의 특정 접근자에 종속됨.
        
        ⇒ 도메인 객체의 내용이 바뀌면 인터페이스도 바뀌어야야함. ( Category, Month 같은 느낌 )
        

    일반적으로 **작은 인터페이스를 권장**. → 도메인 객체의 **다양한 내부연산으로 디펜던시 최소화**

    → 너무 작아도 안됨 ⇒ 안티 응집도 문제가 발생. (여러 기능이 분산되어 필요 기능 찾기가 힘듬.)

    # 명시적 API vs 암묵적 API

    `**BankStatementProcessor`**  이 클래스에 다양한 구현이 기대되지 않는다. 즉, 연산만 할 클래스아닌가? 라는 생각이 들고 굳이 인터페이스로 만들 필요가 없다.

    인터페이스로 만드는 이유는 다양한 구현이 가능하게 만듦으로써 유연성을 확보하고 유지보수를 개선하는 방법이다. 하지만 특별히 이 클래스는 다양한 모습으로 변화할 수 없다. 단순한 입출금 내역에서의 연산을 담당하는 클래스이다.

    위에서 살펴봤던 `**findTransactions()`** 과 같은 메서드를 정의할 수 있는 상황에서 **`findTransactionsGreaterThanEqual()`** 처럼 구체적으로 메서드를 정의해야할까?

    이 문제가 명시적 API vs 암묵적 API 제공 문제다. 양측 모두 장단점이 있다.

    |  | 명시적 API | 암묵적 API |
    | --- | --- | --- |
    | 장점 | 자체적으로 어떤 동작을 하는지 잘 설명이 되어있고 사용하기 쉽다. | 거래내역 검색의 모든 상황에 단순 API로 처리할 수 있다. |
    | 단점 | 메서드의 용도가 특정 상황에 국한되어 각 상황에 맞는 새로운 메서드를 많이 만들어야 한다. | 처음 사용하기가 어렵고, 문서화를 잘해야한다. |

    ⇒ 이 둘은 필요한 질문의 종류에 따라 달라진다. **`findTransactionsGreaterThanEqual()`** 가 가장 흔히 사용하는 연산이라면 명시적 API로 만드는 것이 합리적이다.

    ```java
            // 암묵적인 API를 이용하여 만든 명시적인 API
            public double summarizeTransactions(final BankTransactionsSummarizer bankTransactionsSummarizer) {
            double result = 0;
            for (final BankTransaction bankTransaction : bankTransactionList) {
                result = bankTransactionsSummarizer.summarize(result, bankTransaction);
            }
            return result;
        }

        public double calculateTotalAmount() {
            return summarizeTransactions((acc, bankTransaction) -> acc + bankTransaction.getAmount());
        }

        public double calculateTotalInMonth(final Month month) {
            return summarizeTransactions((acc, bankTransaction) -> bankTransaction.getDate().getMonth() == month
                    ? acc + bankTransaction.getAmount() : acc);
        }

        public double calculateTotalForCategory(final String category) {
            return summarizeTransactions((acc, bankTransaction) -> Objects.equals(bankTransaction.getDescription(), category)
                    ? acc + bankTransaction.getAmount() : acc);
        }
    ```

    # 도메인 클래스 vs 원싯값

    원싯값은 반환하는 것은 일반적으로 좋은 방법이 아니다. 이는 다양한 결과를 반화할 수 없어 유연성이 떨어진다. → double을 감싸는 새 도메인 클래스 `**Summary**`를 만들어 보자

    - 새 클래스에 필요한 필드와 결과를 언제든 추가할 수 있다.
    - 도메인의 다양한 개념 간의 결합을 줄인다.
    - 요구 사항이 바뀔 때 연쇄적으로 코드가 바뀌는 일도 줄인다.

    ```java
    public class Summary {
        private final double amount;

        public Summary(double amount) {
            this.amount = amount;
        }

        public double getAmount() {
            return amount;
        }
    }

    ---

    public Summary calculateTotalAmount() {
            return new Summary(summarizeTransactions((acc, bankTransaction) -> acc + bankTransaction.getAmount()));
        }

        public Summary calculateTotalInMonth(final Month month) {
            return new Summary(summarizeTransactions((acc, bankTransaction) -> bankTransaction.getDate().getMonth() == month
                    ? acc + bankTransaction.getAmount() : acc));
        }

        public Summary calculateTotalForCategory(final String category) {
            return new Summary(summarizeTransactions((acc, bankTransaction) -> Objects.equals(bankTransaction.getDescription(), category)
                    ? acc + bankTransaction.getAmount() : acc));
        }
    ```

    # 다양한 형식으로 내보내기

    OCP와 인터페이스를 사용해서 어떤 조건에도 원하는 결과를 검색할 수 있게 만들었다. 

    먼저 내보낼 `**Exporter` 라는 인터페이스를 정의해서 결합도를 낮춰보자. JSON, XML 뭐든 인터페이스를 구현해서 만들 수 있어 다양한 내보내기를 편리하게 구현할 수 있다.**

    export를 void 반환형으로 만들면 될까? 좋지 않다.

    - void는 아무 도움이 안되고 메서드가 아무것도 안반환해서 타인이 기록하거나 화면에 출력할 가능성이 커진다. 인터페이스로부터 얻을 수 있는 정보가 아무것도 없다.
    - void를 반환하면 테스트도 매우 어렵다.

    ```java
    public interface Exporter {
        String export(SummaryStatistics summaryStatistics);
    }

    ---

    public class HtmlExporter implements Exporter{
        @Override
        public String export(SummaryStatistics summaryStatistics) {
            String result = "<!doctype htmp>";
            result += "<html lang='en'>";
            result += "<head><title>Bank Transaction Report</title></head>";
            result += "<body>";
            result += "<ul>";
            result += "<li><strong>The sum is</strong> : " + summaryStatistics.getSum() + "</li>";
            result += "<li><strong>The average is</strong> : " + summaryStatistics.getAverage() + "</li>";
            result += "<li><strong>The max is</strong> : " + summaryStatistics.getMax() + "</li>";
            result += "<li><strong>The min is</strong> : " + summaryStatistics.getMin() + "</li>";
            result += "</ul>";
            result += "</body>";
            result += "</html>";

            return result;
        }
    }
    ```

    → 테스트 하기도 쉽고 인터페이스로부터 정보를 얻을 수 있게 만드는 것이 좋다.

    # 예외 처리

    잘못되었을 때를 살펴보지 않았다. 파싱하다 잘못되거나 파일을 읽어오지 못하거나 램, 저장 공간이 부족한 경우 → 오류 메시지가 나타나야한다.

    자바는 예외를 일급 언어 기능으로 추가하고 다음과 같은 장점을 제공

    - 문서화 : 메서드 시그니처 자체에 예외를 지원
    - 형식 안정성 : 개발자가 예외 흐름을 처리하고 있는지를 형식 시스템이 파악
    - 관심사 분리 : 비즈니스 로직과 예외 회복 (recovery)이 각각 try/catch 블록으로 구분.

    ⇒ 다만 예외 기능으로 복잡성이 증대됨.

    자바의 예외는 check Exception = 회복해야 하는 대상의 예외다. 던질 수 있는 checked exception 목록을 선언해야함. 혹은 try/catch

    unchecked exception은 프로그램 실행 중 언제든 발생할 수 있는 종류의 예외다. 이전과 달리 메서드 시그니처에 명시적으로 오류를 선언하지 않으면 호출자도 이를 꼭 처리할 필요가 없다.

    그렇다면 검증 로직을 어디 추가하는 것이 좋을까? BankStatement 객체를 생성하는 곳에 추하할 수 있다. 하지만 전용 Validator 클래스를 만드는 것을 권한다.

    - 검증 로직의 코드 중복을 줄인다.
    - 시스템의 다른 부분도 같은 방법으로 검증 가능하다.
    - 로직을 독립적으로 유닛 테스트할 수 있다.
    - 프로그램 유지보수와 이해하기 쉬운 SRP를 따른다.

    검증또한 마찬가지로 과도한 세밀함은 코드 복잡을 야기한다. 그렇다고 IllegalArgumentException 등의 미확인 예외로 모든 예뢰를 지정해도 안된다 → 구체적인 리커버리 로직을 만들 수 없다.

    이를 해결하기 위한 Notfication 패턴이 존재한다.

    ```java
    public class Notifiaction {
        private final List<String> errors = new ArrayList<>();

        public void addError(final String message) {
            errors.add(message);
        }
        
        public boolean hasErrors() {
            return !errors.isEmpty();
        }
        
        public String errorMessage() {
            return errors.toString();
        }
        
        public List<String> getErrors() {
            return this.errors;
        }
    }
    ```

    위처럼 예외가 발생하면 예외를 던지는 것이 아니라 Notification 객체에 메시지를 추가한다.

    # 예외 사용 가이드라인

    - 예외 무시하지 않기
        - 문제의 근본 원인을 알 수 없다고 예외 무시 X
        - 처리방법이 명확하지 않으면 미확인 예외를 대신 던진다.
    - 일반적 예외는 잡지 않음
        - Exception, RuntimeException과 달리 구체적으로 예외를 잡으면 가독성이 높아지고 더 세밀한 예외처리가 가능하다.
    - 예외 문서화
        
        ```java
        @throws NosuchFileException 파일이 존재하지 않은 경우
        @throws DirectoryNotEmptyException 파일이 디렉터리고 비어있지 않아 삭제불가 할 때
        @throws IOException I/O 오류가 발생했을때
        ```
        
        - 특정 API 수준에서 문서화하자.
    - 특정 구현에 종속된 예외를 던지면 API의 캡슐화가 깨진다 주의하자.
        - OracleException → 오라클에 종속됨 (코드도 같이 종속됨)
    - 예외로 흐름을 제어하지 말자.
        - 예외를 이용하여 특정 로직의 분기를 처리하면 안된다. 그리고 정말 예외를 던져야 하는 상황이 아니라면 예외를 만들지 않아야 한다.
            
            → 예외 발생 시 ⇒ 스택 트레이스 생성, 보존과 관련된 부담 발생.



</details>

<details>
    <summary> 세준 </summary>

- 세준
    
    ## 기억에 남는 내용
    
    ### 예외를 처리하기 위한 판단 기준 및 주의점, 몇가지 예외 처리 방안
    
    체크드 예외와 언체크드예외의 판단 기준
     → 
    
    ### 묵시적 API 와 명시적 API
    
    추상도가 높은 API vs 구체적인 API
    
    ### 안티 응집도
    
    지나치게 세분화된 인터페이스는 오히려 코드의 유지보수를 해침
    
    ### 도메인 객체를사용하는 이유
    
    추상화를 통해 요구사항이 변경됐을때 변경되어야하는 코드의 수를 줄일 수 있음(내부 구현만 수정하면됨)




</details>

<details>
    <summary> 소현 </summary>
    
- 소현
    # 입출금 내역 분석기 확장판

    ### 개방/폐쇄 원칙

    : 코드를 직접 바꾸지 않고 해당 메서드나 클래스의 동작을 바꿀 수 있다.

    새로운 인터페이스를 이용해 반복 로직과 비즈니스 로직의 결합을 제거할 수 있다.

    <aside>
    💡 **함수형 인터페이스**
    한 개의 추상 메서드를 포함하는 인터페이스. `@FunctionalInterface` 어노테이션으로 인터페이스의 의도를 더 명확하게 표현할 수 있다.

    </aside>

    기존 메서드의 바디를 바꿀 필요 없이 새로운 구현을 인수로 전달한다.

    → 변경 없이도 확장성은 개방된다.

    - 기존에 이미 구현하고 검증한 코드를 바꾸는 일을 최소화할 수 있다.
    - 예전 코드를 바꾸지 않고 새로운 기능을 추가할 수 있다.
    - 기존 코드를 바꾸지 않으므로 기존 코드가 잘못될 가능성이 줄어든다.
    - 코드가 중복되지 않으므로 기존 코드의 재사용성이 높아진다.
    - 결합도가 낮아지므로 코드 유지보수성이 좋아진다.

    ### 갓 인터페이스

    모든 헬퍼 연산이 명시적인 API 정의에 포함되면서 인터페이스가 복잡해진다.

    자바의 인터페이스는 모든 구현이 지켜야 할 규칙을 정의한다. 즉 구현 클래스는 인터페이스에서 정의한 모든 연산의 구현 코드를 제공해야 한다. 따라서 인터페이스를 바꾸면 이를 구현한 코드도 바뀐 내용을 지원하도록 갱신되어야 한다. 더 많은 연산을 추가할수록 더 자주 코드가 바뀌며, 문제가 발생할 수 있는 범위도 넓어진다.

    인터페이스가 도메인 객체의 특정 접근자에 종속되면, 도메인 객체의 세부 내용이 바뀔 때 인터페이스도 바뀌어야 하며 결과적으로 구현 코드도 바뀌어야 한다.

    → 작은 인터페이스 권장

    ### 지나친 세밀함

    인터페이스가 지나치게 세밀함(작음) → 안티 응집도

    기능이 여러 인터페이스로 분산되면 필요한 기능을 찾기 어렵다. (유지보수성 저하)

    복잡도가 높아지고, 새로운 인터페이스가 계속해서 추가될 수밖에 없다.

    ### 명시적 API vs 암묵적 API

    ```java
    findTransactionGreaterThanEqual()
    ```

    - 자체적으로 어떤 동작을 수행하는지 잘 설명되어 있다.
    - 사용하기 쉽다.
    - API의 가독성을 높이고, 쉽게 이해하도록 메서드 이름이 서술적이다.
    - 용도가 특정 상황에 국한된다면 각 상황에 맞는 새로운 메서드를 많이 만들어야 한다는 단점이 있다.

    ```java
    findTransactions()
    ```

    - 처음 사용하기가 어렵고, 문서화를 잘해놓아야 한다.
    - 공통으로 필요한 모든 상황을 단순한 API로 처리할 수 있다.

    ⇒ 정해진 방법X, 다만 명시적 API가 가장 흔히 사용되는 것이면 쉽게 이해하도록 명시적으로 사용하는 것이 바람직하고 그렇지 않다면 암묵적 API를 사용하는 것이 바람직하다.

    ### 적절한 인터페이스 정의와 구현

    ```java
    package com.study.rwfd.week3;

    public interface Exporter {
        void export(SummaryStatistics summaryStatistics);
    }
    ```

    `void() 메서드`

    → 아무 도움도 되지 않고, 기능을 파악하기도 어렵다.

    → void 를 반환하면 assertion 으로 결과를 테스트하기 어렵다.

    ```java
    package com.study.rwfd.week3;

    public interface Exporter {
        String export(SummaryStatistics summaryStatistics);
    }
    ```

    정의를 내보내는 API 정의 → 인터페이스를 준수하는 다양한 기능을 구현할 수 있고, 테스트 할 수도 있다.

    ### 예외 처리

    - 문서화 : 메서드 시그니처 자체에 예외를 지원한다.
    - 형식 안전성 : 개발자가 예외 흐름을 처리하고 있는지를 형식 시스템이 파악한다.
    - 관심사 분리 : 비즈니스 로직과 예외 회복이 각각 try/catch 블록으로 구분된다.
    - 복잡성이 증가하는 단점이 있다.

    대다수의 예외를 미확인 예외로 지정하고 꼭 필요한 상황에서만 확인된 예외로 지정해 불필요한 코드를 줄여야 함.

    ### 예외 사용 가이드라인

    - 예외를 무시하지 않음
    - 일반적인 예외는 잡지 않음
    - 예외 문서화
    - 특정 구현에 종속된 예외를 주의할 것
    - 예외 vs 제어 흐름

    ### 빌드 도구

    - 프로젝트에 적용되는 공통적인 구조를 제공하기 때문에 동료 개발자가 프로젝트를 좀 더 편하게 받아들인다.
    - 응용프로그램을 빌드하고 실행하는 반복적이고 표준적인 작업을 설정한다.
    - 저수준 설정과 초기화에 들이는 시간을 절약하므로 개발에만 집중할 수 있다.
    - 잘못된 설정이나 일부 빌드 과정 생략 등으로 발생하는 오류의 범위를 줄인다.
    - 공통 빌드 작업을 재사용해 이를 다시 구현할 필요가 없으므로 시간을 절약한다.

</details>

<details>
    <summary> 얘기해본 내용 정리 </summary>

- 얘기해본 내용 정리
    # 토론 내용

    ## 암묵적 API 명시적 API

    자주 쓰는건 명시적 API로 하는거 아닌가? 일단 나는 명시적 API로 과제 짰음

    캠슐화와 상관이 있는 내용인가? 상관 없는 내용같다.

    암묵적 API를 잘 모르겠다.

    다형성과 관련된 내용인가 ? 그런거 같다.

    기본적으로는 암묵적으로 하는게 좋지만(좋은 문서화와 함께), 많이 쓰는거면 명시적 API로 제공해라

    ## 책에서 말하는 도메인 클래스를 쓰는게 좋은 이유

    도메인 클래스 vs 원싯값
    책을 읽지 않았을 때 메소드의 반환은? 원시값 즉시 반환~~

    - 도메인 클래스에 다른 필드나 결과값이 필요하면 손쉽 추가가 가능하다.
    - 도메인의 다양한 개념 간의 결합을 줄인다.
        - double만 봤을때 어떤 개념인지 파익이 힘들다?
        (**double 가질 수 있는 의미가 많기 때문에** 어떤 개념인지 파악하기 힘들다.
        도메인 클래스로 감싸면 이를 쉽게 할 수 있다)
    - 요구사항의 변경에 유연하게 대처할 수 있다.(내부 구현만 바꾸면 되므로)
        - 다양한 형식으로 내보내기를 하기 편하다

    ## 다양한 형식으로 내보내기

    - 숫자로 내보낼떄
        - 간단한 구현, 요구사항의 변경에 따른 요구사항 변경 힘듦
    - 컬렉션
        - 유연성은 좋아지지만 오로지 컬렉션만 반환해야 함
    - 특별한 도메인 객체
        - 기존의 코드 변경 없이 도메인 클래스의 내부 구현으로 요구사항의 변경에 대응할 수 있다.
    - 더 복잡한 도메인 객체
        - 어렵다….. 더 찾아봐야 할듯

    ## 인터페이스의 반환형이 void 면?

    1. 기능파악이 어렵다.(인터페이스로 부터 정보를 얻을 수 없기 떄문)
    2. Assertions로 테스트를 하기가 어렵다. (아무것도 알려주지 않기때문에)
        1. 과제에서 delete의 반환형은 어떤게 좋은가?
            1. 다들 void 로 해놨다.
            2. 찾아보니까 int 나 boolean으로 많이 하더라

    ## 예외처리

    예외처리의 장점

    - 문서화
        - 메서드 시그니처에 이 메소드가 어떤 예외를 뱉을 수 있는지 알려줄 수 있음(사용자가 사용하기 쉬움)
    - 형식 안전성
        - 에러가 처리의 흐름을 강제한다.(checked Exception)
    - 관심사 분리
        - 비지니스 로직과의 분리가 가능하다
        - 과거에는 if문으로 전부 때렸는데 예외처리로 비지니스 로직과 명확한 분리가 가능해진다.

    ## 체크드 예외와 언체크드 예외 선택

    어지간하면 UnChecked Exception을 쓰는게 맞다. 

    ## Validator의 분리

    책은 일반적으로 분리하는게 좋다고 나온다.

    - 검증로직 재사용
    - 로직의 독립적인 유닛 테스트 할 수 있다.
    - 유지보수와 이해하기 쉬운 SRP

    객체에 대한 validate 객체 내부에서 한다.

    들어오는 요청에 대한 validate는 요청을 받는 곳에서 한다.

    ## 과도하게 세밀한 예외처리

    예외가 너무 세밀하다…!

    → 너무 많은 설정 작업

    → 생산성이 현저하게 떨어짐(사용자가 API를 쉽게 사용할 수없음)

    ## 과도하게 덤덤한 예외처리

    예외가 너무 덤덤하다…!

    → 전부 동일한 예외로 인해 구체적인 회복 로직을 만들 수 없음

    공통 문제점

    → 여러 오류가 발생했을 때 오류 목록을 모아 사용자에게 제공할 수 없음 (노티피케이션으로 해결)


</details>