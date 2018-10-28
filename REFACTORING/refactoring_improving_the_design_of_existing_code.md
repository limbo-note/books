- [1. A First Example](#1-a-first-example)
	- [1.2 The First Step in Refactoring](#12-the-first-step-in-refactoring)
	- [1.3 Decomposing and Redistributing the 'Statement' Method](#13-decomposing-and-redistributing-the-statement-method)
- [2. Principles in Refactoring](#2-principles-in-refactoring)
	- [2.1 Defining Refactoring](#21-defining-refactoring)
	- [2.2 Why Refactor?](#22-why-refactor)
	- [2.3 When Refactor?](#23-when-refactor)
	- [2.5 Problems with refactoring](#25-problems-with-refactoring)
	- [2.6 Refactoring and Design](#26-refactoring-and-design)
	- [2.7 Refactoring and Performance](#27-refactoring-and-performance)
- [3. Bad Smells in Code](#3-bad-smells-in-code)
- [4. Building Tests](#4-building-tests)
	- [4.1 Self-testing](#41-selftesting)
	- [4.2 The JUnit Testing Framework](#42-the-junit-testing-framework)
	- [4.3 Adding More Tests](#43-adding-more-tests)
- [5. Format:](#5-format)
- [6. Composing Method](#6-composing-method)
	- [6.1 Extract Method](#61-extract-method)
	- [6.2 Inline Method](#62-inline-method)
	- [6.3 Inline Temp](#63-inline-temp)
	- [6.4 Replace Temp with Query](#64-replace-temp-with-query)
	- [6.5 Introduce Explaining Variable](#65-introduce-explaining-variable)
	- [6.6 Split Temporary Variable](#66-split-temporary-variable)
	- [6.7 Remove Assignments to Parameters](#67-remove-assignments-to-parameters)
	- [6.8 Replace Method with Method Object](#68-replace-method-with-method-object)
	- [6.9 Substitute Algorithm](#69-substitute-algorithm)
- [7. Moving Features Between Objects](#7-moving-features-between-objects)
	- [7.1 Move Method](#71-move-method)
	- [7.2 Move Field  ](#72-move-field-)
	- [7.3 Extract Class](#73-extract-class)
	- [7.4 Inline Class](#74-inline-class)
	- [7.5 Hide Delegate](#75-hide-delegate)
	- [7.6 Remove Middle Man](#76-remove-middle-man)
	- [7.7 Introduce Foreign Method](#77-introduce-foreign-method)
	- [7.8 Introduce Local Extension](#78-introduce-local-extension)
- [8. Organizing Data](#8-organizing-data)
	- [8.1 Self Encapsulate Field](#81-self-encapsulate-field)
	- [8.2 Replace Data Value with Object](#82-replace-data-value-with-object)
	- [8.3 Change Value to Reference](#83-change-value-to-reference)
	- [8.4 Change Reference to Value](#84-change-reference-to-value)
	- [8.5 Replace Array with Object](#85-replace-array-with-object)
	- [8.6 Duplicate Observed Data](#86-duplicate-observed-data)
	- [8.7 Change Unidirectional Association to Bidirectional](#87-change-unidirectional-association-to-bidirectional)
	- [8.8 Change Bidirectional Association to Unidirectional](#88-change-bidirectional-association-to-unidirectional)
	- [8.9 Replace Magic Number with Symbolic Constant](#89-replace-magic-number-with-symbolic-constant)
	- [8.10 Encapsulate Field](#810-encapsulate-field)
	- [8.11 Encapsulate Collection(Arrays)](#811-encapsulate-collectionarrays)
	- [8.12 Replace Record with Data Class](#812-replace-record-with-data-class)
	- [8.13 Replace Type Code with Class](#813-replace-type-code-with-class)
	- [8.14 Replace Type Code with Subclasses](#814-replace-type-code-with-subclasses)
	- [8.15 Replace Type Code with State/Strategy](#815-replace-type-code-with-statestrategy)
	- [8.16 Replace Subclass with Fields](#816-replace-subclass-with-fields)
- [9. Simplifying Conditional Expressions](#9-simplifying-conditional-expressions)
	- [9.1 Decompose Conditional ](#91-decompose-conditional-)
	- [9.2 Consolidate Conditional Expression](#92-consolidate-conditional-expression)
	- [9.3 Consolidate Duplicate Conditional Fragments](#93-consolidate-duplicate-conditional-fragments)
	- [9.4 Remove Control Flag](#94-remove-control-flag)
	- [9.5 Replace Nested Conditional with Guard Clauses](#95-replace-nested-conditional-with-guard-clauses)
	- [9.6 Replace Conditional with Polymorphism](#96-replace-conditional-with-polymorphism)
	- [9.7 Introduce Null Object](#97-introduce-null-object)
	- [9.8 Introduce Assertion](#98-introduce-assertion)
- [10. Making Method Calls Simpler](#10-making-method-calls-simpler)
	- [10.1 Rename Method](#101-rename-method)
	- [10.2 Add Parameter](#102-add-parameter)
	- [10.2 Remove Parameter](#102-remove-parameter)
	- [10.3 Separate Query from Modifier](#103-separate-query-from-modifier)

**refactoring:** does not alter the external behavior of the code yet improves its internal structure

# 1. A First Example

	public class Movie {  
    
     public static final int  CHILDRENS = 2;  
     public static final int  REGULAR = 0;  
     public static final int  NEW_RELEASE = 1;  
    
     private String _title;  
     private int _priceCode;  
    
     public Movie(String title, int priceCode) {  
         _title = title;  
         _priceCode = priceCode;  
     }  
                                                  
     public int getPriceCode() {  
         return _priceCode;  
     }  
    
     public void setPriceCode(int arg) {  
       _priceCode = arg;  
     }  
    
     public String getTitle (){  
         return _title;  
     };  
  	}

	class Rental {  
       private Movie _movie;  
       private int _daysRented;  
    
       public Rental(Movie movie, int daysRented) {  
         _movie = movie;  
         _daysRented = daysRented;  
       }  
       public int getDaysRented() {  
         return _daysRented;  
       }  
       public Movie getMovie() {  
         return _movie;  
       }   
  	}  

	class Customer {  
     private String _name;  
     private Vector _rentals = new Vector();  
    
     public Customer (String name){  
         _name = name;  
     };  
    
     public void addRental(Rental arg) {  
       _rentals.addElement(arg);  
     }  
     public String getName (){  
         return _name;  
     }
	 
	 public String statement() {  
            double totalAmount = 0;  
            int frequentRenterPoints = 0;  
            Enumeration rentals = _rentals.elements();  
            String result = "Rental Record for " + getName() + "\n";  
            while (rentals.hasMoreElements()) {  
                double thisAmount = 0;  
                Rental each = (Rental) rentals.nextElement();  
      
                //determine amounts for each line  
                switch (each.getMovie().getPriceCode()) {  
                    case Movie.REGULAR:  
                        thisAmount += 2;  
                        if (each.getDaysRented() > 2)
                            thisAmount += (each.getDaysRented() - 2) * 1.5;  
                        break;  
                    case Movie.NEW_RELEASE:  
                        thisAmount += each.getDaysRented() * 3;  
                        break;  
                    case Movie.CHILDRENS:  
                        thisAmount += 1.5;  
                        if (each.getDaysRented() > 3)  
                            thisAmount += (each.getDaysRented() - 3) * 1.5;  
                        break;  
      
                }  
      
                // add frequent renter points  
                frequentRenterPoints ++;  
                // add bonus for a two day new release rental  
                if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE)  &&  
    each.getDaysRented() > 1) frequentRenterPoints ++;  
      
                //show figures for this rental  
                result += "\t" + each.getMovie().getTitle()+ "\t" +  
    String.valueOf(thisAmount) + "\n";  
                totalAmount += thisAmount;  
      
            }  
            //add footer lines  
            result +=  "Amount owed is " + String.valueOf(totalAmount) + "\n";  
            result += "You earned " + String.valueOf(frequentRenterPoints) +  
    " frequent renter points";  
            return result;    
    }
  	}  

The example code presented above is ugly and not well-designed, it's hard to change.

The statement method is where the changes have to be made to deal with changes in **classification** and charging **rules**. If, however, we copy the statement to an HTML statement, we need to ensure that any changes are completely consistent.

**When you find you have to add a feature to a program, and the program's code is not structured in a convenient way to add the feature, first refactor the program to make it easy to add the feature, then add the feature.**
    

### 1.2 The First Step in Refactoring
Build a solid set of tests for the section of code.(And **self-checking** is important)

### 1.3 Decomposing and Redistributing the 'Statement' Method
refactoring changes the programs in small steps, so you can easily find bugs.

write code that humans can understand.

In most cases a method should be on the object whose data it uses.(高内聚)

try to eliminate temporary variables, usually can be extracted to a method.

change 'switch' to polymorphism

**state pattern**:
1. replace type code with state/strategy
2. move the switch statement into the price(state) class
3. replace conditional with polymorphism

**Extract Method, Move Method, Replace Conditional with Polymorphism**

**test->small change->test->small change ....**

# 2. Principles in Refactoring
### 2.1 Defining Refactoring
**Refactoring(noun):** a change made to the internal structure of software to make it easier to understand and cheaper to modify without changing its observable behavior.

**Refactor(verb):** to restructure software by applying a series of refactorings without changing its observable behavior.

### 2.2 Why Refactor?
**Refactoring improves the design of software:** regular refactoring helps code retain its shape; an important aspect of improving design is to eliminate duplicate code.

**Refactoring makes software easier to understand**

**Refactoring helps you find bugs**

**Refactoring helps you program faster**

### 2.3 When Refactor?
**The Rule of three:** first time, just do; second time, do the same thing, try not to duplicate; third time, do the same thing, you should refactor.

**Refactor when add function(feature)**

### 2.5 Problems with refactoring
**Databases:** applications are tightly coupled to the database schema, and database is difficult to change.

**Changing Interfaces:** published method - you cannot change the callers' code. When refactoring these methods, keep the old one, and just let it call the new one.(And mark the old one as 'deprecated'); **protecting interfaces**; **adding an exception to the throws clause**

**When shouldn't refactor:**  
when the existing code is such a mess, it's eaiser to start from beginning.  
when the code is full of bugs, you should fix bugs before refactoring.  
when you are close to a deadline.  

### 2.6 Refactoring and Design
### 2.7 Refactoring and Performance
it slows the software in the short term while refactoring, but makes the software easier to tune during optimization.

# 3. Bad Smells in Code
- duplicated code. try to unify them.
	- Extract Method
	- Pull Up Field
	- Form Template Method
	- Substitute Algorithm
	- Extract Class
- long method. A method should be short, and has a good name. **Whenever need to comment something, write a method instead**
	- Extract Method
	- Replace Temp with Query.(too many local temps)
	- Introduce Parameter Object.(too many parameters)
	- Preserve Whole Object.(too many parameters)
	- Replace Method with Method Object
	- Decompose Conditional
- large class. A class doing too much often has too many instance variables, which usually results in duplicated code.
	- Extract Class. (解决低内聚)
	- Extract Subclass. (解决低内聚)
	- Extract Interface. (determine how clients use the class)
	- Duplicate Observed Data. (for GUI class)
- long parameter list. (try to unify some parameters into class)
	- Replace Parameter with Method. (if you can get parameter data from method of the class you already get)
	- Perserve Whole Object 
	- Introduce Parameter Object
- divergent change. (单一权责原则)
	- Extract Class
- shotgun surgery. (与单一权责相反，不是一个类有多个修改的理由，是一个修改的理由会影响到很多的类)
	- Move Method
	- Move Field
	- Inline Class
- feature envy. (a class uses too much data from other classes)
	- Move Method
- data clumps. (the same clumps of data appear in lots of places) try to extract them as class. (After extracting, consider whether there is feature envy, if so, move original method to the new class)
- primitive obsession. try to use objects rather than primitive type.
	- Replace Data Value with Object
	- Replace Type Code with Class
	- Replace Type Code with Subclasses
	- Replace Type Code with State/Strategy
	- Replace Array with Object
- switch statements. 
	- Replace Conditional with Polymorphism
	- Replace Parameter with Explicit Methods
- parallel inheritance hierarchies. (may cause shotgun surgery)
- lazy class
- speculative generality. (try not to speculate future requirements and not to hook unnecessary funtions/code that are not required)
- temporary field. (field only be set in certain circumstances)
- message chains. ( t = A.getB().getC().getD() .... )
	- Hide Delegate
	- Extract Method
	- Move Method
- middle man (....)
- inappropriate intimacy. (delve too much in other classes)
- alternative classes with different interfaces.
- incomplete library class.
- public fields in data class.
	- Encapsulate Field
	- use Move Method to move behavior into data class
- refused bequest. (....)
- comments. when you feel the need to write a comment, first try to refactor the code so that any conmment becomes superfluous.

# 4. Building Tests

### 4.1 Self-testing
(it would just print 'OK' to the screen if all was well)

**Make sure all tests are fully automatic and that they check their own results**  
**A suite of tests can save a lot of time finding bugs(frequent testing)**  
**write tests code before function code**  

### 4.2 The JUnit Testing Framework

Example:
	
	{
	}

When you get a bug report, start by writing a unit test that exposes the bug.  

### 4.3 Adding More Tests
only test areas that that you are most worried about.

How JUnit Work?:

- run each of suite's component tests
- each test-case execute setUp -> the body of tested method -> tearDown

build a test suite that contains a test-case for every method that starts with 'test': `junit.textui.TestRunner.run(new TestSuite(XXXTester.class));`

Think of the boundary conditions under which things might go wrong and concentrate your tests there.

Don't forget to test the exceptions which are expected.

# 5. Format:
- name
- summary
- motivation
- mechanics
- examples
# 6. Composing Method
### 6.1 Extract Method
summary:  
turn the fragment into a method whose name explains the purpose of the method.  

motivation:  
(have a code fragment can be grouped together)  
a method is too long / need a comment, short well-named methods are prefered.

mechanics:  
>- create method, name it after intention
>- extract body into new method
>- scan local variables, temporary variables
>- Split Temporary Variable / Replace Temp with Query
>- local variables -> parameter

example:  
>- no local variables
>- using local variables
>- reassigning a local variable

### 6.2 Inline Method
summary:  
put the method's body into the body of the callers  

motivation:  
(a method's body is just as clear as its name)  
or when you have a group of methods that seem badly factored. Inline them together and reextract.

mechanics:  
>- check if polymorphic, don't inline if subclasses override this method
>- find all calls
>- replace calls with body

### 6.3 Inline Temp
summary:  
replace all references to temp with the expression  

motivation:  
most when *Replace Temp with Query*. You have a temp that is assigned to once with a simple expression, and the temp is getting in the way of other refactorings.

mechanics:   
>- make sure it's assigned only once
>- find all references, replace with the right-hand

### 6.4 Replace Temp with Query
summary:  
extract the expression into a method. Replace all references to the temp with the method.

motivation:  
when using a temporary variable to hold the result of an expression. A vital step before *Extract Method*.

mechanics:  
>- look for a temp assigned only once. if not, consider *Split Temporary Variable*
>- Extract the right-hand side of the assignment into a method. Make sure it's free of side effects(does not
modify any object). if not, consider *Separate Query
from Modifier.*
>- replace the temp with method.

### 6.5 Introduce Explaining Variable
summary:  
put the result of the expression, or parts of the expression, in a temporary variable with a name

motivation:  
(You have a complicated expression.)  
expressions complex and hard to read. Much time can use *Extract Method* instead, expect when it's hard to deal with local variables.  

mechanics:  
>- declare a final temp, set it to the result of expression
>- replace

consider *Extract Method* 

### 6.6 Split Temporary Variable
summary:  
make a different temp for each assignment
	
	double temp = 2 * (_height + _width);
    System.out.println (temp);
    temp = _height * _width;
    System.out.println (temp);
↓

	final double perimeter = 2 * (_height + _width);
    System.out.println (perimeter);
    final double area = _height * _width;
    System.out.println (area);

motivaiton:  
temp variable assigned to more than once, which is not a loop/collecting variable

using a temp for two different things is very confusing for the reader.
	
### 6.7 Remove Assignments to Parameters
summary:  
The code assigns to a parameter. Use a temporary variable instead.

	int discount (int inputVal, int quantity, int yearToDate) {
    	if (inputVal > 50) inputVal -= 2;
		return inputVal;
	}
↓

	int discount (int inputVal, int quantity, int yearToDate) {
    	int result = inputVal;
    	if (inputVal > 50) result -= 2;
    	return result;
    }

### 6.8 Replace Method with Method Object
summary:  
Turn the method into its own object so that all the local variables become fields on that object.  
You can then decompose the method into other methods on the same object.

motivation:  
You have a long method that uses local variables in such a way that you cannot apply *Extract Method*.  
the local temps are rampant, so it's difficult to decompose method into small methods.

mechanics:  
>- create class after original method
>- each temp, parameter of original method -> a field in new class.
>- original class -> a new final field in new class
>- method parameter, original class -> constructor parameter
>- a new method in class named 'compute'
>- original body -> 'compute' body
>- then you can decompose the 'compute' method

### 6.9 Substitute Algorithm
summary:  
Replace the body of the method with the new algorithm.

motivation:  
You want to replace an algorithm with one that is clearer.

mechanics:  
>- replace
>- test, if OK, finished
>- if not OK, debug

# 7. Moving Features Between Objects
### 7.1 Move Method
summary:  
Create a new method with a similar body in the class it uses most.   

motivations:  
used by another class more than class on which it's defined.  

### 7.2 Move Field  
summary:  
Create a new field in the target class, and change all its users. 

motivations:  
used by another class more than the class on which it is defined.   
when doing *Extract Class*  

### 7.3 Extract Class
summary:  
Create a new class and move the relevant fields and methods from the old class into the new class.

motivations:  
have one class doing work that should be done by two.  
A class that is too big to understand easily.  

*Extract Class* is a common technique for improving the liveness of a concurrent program because it allows you to have separate locks on the two resulting classes.

### 7.4 Inline Class
summary:  
Move all its features into another class and delete it. 

motivations:  
A class isn't doing very much.  
the reverse of *Extract Class*.   

### 7.5 Hide Delegate
summary:  
Create methods on the server to hide the delegate.  

motivations:  
A client is calling a delegate class of an object.  

example:  

	class Person {
        Department _department;
        public Department getDepartment() {
            return _department;
        }
        public void setDepartment(Department arg) {
            _department = arg;
        }
    }

    class Department {
        private String _chargeCode;
        private Person _manager;
        public Department (Person manager) {
            _manager = manager;
        }
        public Person getManager() {
        	return _manager;
    	}
	...
if get a person's manager:  
	
	manager = john.getDepartment().getManager();

this reveals to the client how the department class works, not encapsulated.  
↓

	public Person getManager() {
		return _department.getManager();
	}

### 7.6 Remove Middle Man
summary:  
Get the client to call the delegate directly.  

motivaitons:  
A class is doing too much simple delegation, and *Hide Delegate* becomes painful.  

machanics:  
the reverse of *Hide Delegate*. 

### 7.7 Introduce Foreign Method
summary:  
Create a method in the client class with an instance of the server class as its first argument.  

motivations:  
A server class you are using needs **an** additional method, but you can't modify the class.  

example:  

	Date newStart = new Date (previousEnd.getYear(), previousEnd.getMonth(), previousEnd.getDate() + 1);
↓
	
	Date newStart = nextDay(previousEnd);
    private static Date nextDay(Date arg) {
        return new Date (arg.getYear(),arg.getMonth(), arg.getDate() +1);
    }
*Introduce Foreign Method* is just a work-around（紧急措施）, if possible, move the method to server class.  

### 7.8 Introduce Local Extension
summary:  
Create a new class that contains extra methods. Make this extension class a **subclass** or a **wrapper** of the original.  

motivations:  
A server class you are using needs **several** additional methods, but you can't modify the class.  

example:  
>- using a subclass
>- using a wrapper

# 8. Organizing Data
### 8.1 Self Encapsulate Field
summary:  
Create getting and setting methods for the field and use only those to access the field.

motivations:  
accessing a field directly, but the coupling to the field is becoming awkward.  
The most important time to use *Self Encapsulate Field* is when you are accessing a field in a superclass but you want to override this variable access with a computed value in the subclass.  

example:  

	private int _low, _high;
    boolean includes (int arg) {
        return arg >= _low && arg <= _high;
    }
↓
	
	private int _low, _high;
    boolean includes (int arg) {
        return arg >= getLow() && arg <= getHigh();
    }
    int getLow() {return _low;}
    int getHigh() {return _high;}

Use direct variable access as a first resort, until it gets in the
way. Once things start becoming awkward, switch to indirect variable access. 

### 8.2 Replace Data Value with Object
summary:  
Turn data item into an object. 

motivations:  
a data item that needs additional data or behavior.  

### 8.3 Change Value to Reference
summary:  
Turn object into a reference object.

motivations:  
a class with many equal instances that you want to replace with a 
single object.  

**Reference objects** are things like customer or account. Each object stands for one object in the real world, and you use the object identity to test whether they are equal.  
**Value objects** are things like date or money. They are defined entirely through their data values. You don't mind that copies exist.  

usually involve *Replace Constructor with Factory Method*  

### 8.4 Change Reference to Value
the reverse of *Change Value to Reference*  

be careful about the 'equals' method and 'hashcode' method.

### 8.5 Replace Array with Object
summary:  
Replace the array with an object that has a field for each element.

motivations:  
an array in which certain elements mean different things. 

example:  

	String[] row = new String[3];
    row [0] = "Liverpool";
    row [1] = "15";
↓

    Performance row = new Performance();
    row.setName("Liverpool");
    row.setWins("15");

### 8.6 Duplicate Observed Data
when GUI involved
(...)

### 8.7 Change Unidirectional Association to Bidirectional
summary:  
Add back pointers, and change modifiers to update both sets.

motivations:  
two classes that need to use each other's features, but there is only a one-way link.

### 8.8 Change Bidirectional Association to Unidirectional
motivations:  
a two-way association but one class no longer needs features from the other

the reverse of *Change Unidirectional Association to Bidirectional*

### 8.9 Replace Magic Number with Symbolic Constant
summary:  
Create a constant, name it after the meaning, and replace the number with it.  

motivations:  
have a literal number with a particular meaning.  

example:  

	double potentialEnergy(double mass, double height) {
        return mass * 9.81 * height;
    }
↓

    double potentialEnergy(double mass, double height) {
        return mass * GRAVITATIONAL_CONSTANT * height;
    }
    static final double GRAVITATIONAL_CONSTANT = 9.81;

### 8.10 Encapsulate Field
summary:  
Make the public field private and provide accessors.

motivations:  
there is a public field.  

### 8.11 Encapsulate Collection(Arrays)
motivations:  
A method returns a collection.  

summary:  
Make it return a read-only view and provide add/remove methods.

there should not be a setter for collection: rather there should be operations to add and remove elements. 

example:

	class Person{
        public Set getCourses() {
            return _courses;
        }
        public void setCourses(Set arg) {
            _courses = arg;
        }
        private Set _courses;
    }
↓

	class Person{
        public void addCourse (Course arg) {
            _courses.add(arg);
        }
        
        public void removeCourse (Course arg) {
            _courses.remove(arg);
        }

        private Set _courses = new HashSet();

        public Set getCourses() {
            return Collections.unmodifiableSet(_courses);
        }

        public void initializeCourses(Set arg) {
            Assert.isTrue(_courses.isEmpty());
            _courses.addAll(arg);
        }
    }

same when it comes to Arrays

### 8.12 Replace Record with Data Class
turn record into object

### 8.13 Replace Type Code with Class
motivations:  
A class has a numeric type code that **does not affect** its behavior.

example:  

	class Person {
        public static final int O = 0;
        public static final int A = 1;
        public static final int B = 2;
        public static final int AB = 3;
        
		private int _bloodGroup;
        public Person (int bloodGroup) {
            _bloodGroup = bloodGroup;
        }
        public void setBloodGroup(int arg) {
            _bloodGroup = arg;
        }
        public int getBloodGroup() {
            return _bloodGroup;
        }
    }
↓

	class Person {
        public static final int O = BloodGroup.O.getCode();
        public static final int A = BloodGroup.A.getCode();
        public static final int B = BloodGroup.B.getCode();
        public static final int AB = BloodGroup.AB.getCode();
        
        private BloodGroup _bloodGroup;
        public Person (int bloodGroup) {
            _bloodGroup = BloodGroup.code(bloodGroup);
        }
        public int getBloodGroup() {
            return _bloodGroup.getCode();
        }
        public void setBloodGroup(int arg) {
            _bloodGroup = BloodGroup.code(arg);
        }
    }

### 8.14 Replace Type Code with Subclasses
motivations:  
have an immutable type code that **affects** the behavior of a class.

(...)

### 8.15 Replace Type Code with State/Strategy

a similar approach compared with 8.14??????

### 8.16 Replace Subclass with Fields
motivations:  
have subclasses that vary **only** in methods that return **constant** data.  

summary:  
Change the methods to superclass fields and eliminate the subclasses.

example:

	abstract class Person {
        abstract boolean isMale();
        abstract char getCode();
    }

    class Male extends Person {
        boolean isMale() {
            return true;
        }
        char getCode() {
            return 'M';
        }
    }

    class Female extends Person {
        boolean isMale() {
            return false;
        }
        char getCode() {
            return 'F';
        }
    }
↓

	class Person...
        private final boolean _isMale;
        private final char _code;

        protected Person (boolean isMale, char code) {
            _isMale = isMale;
            _code = code;
        }

        boolean isMale() {
            return _isMale;
        }
        char getCode(){
            return _code;
        }

        static Person createMale(){
            return new Person(true, 'M');
        }
    }

# 9. Simplifying Conditional Expressions

### 9.1 Decompose Conditional 
similar to *Extract Method*

example:  

	if (date.before (SUMMER_START) || date.after(SUMMER_END))
        charge = quantity * _winterRate + _winterServiceCharge;
    else charge = quantity * _summerRate;
↓

	if (notSummer(date))
        charge = winterCharge(quantity);
    else charge = summerCharge (quantity);

    private boolean notSummer(Date date) {
        return date.before (SUMMER_START) || date.after(SUMMER_END);
    }
    private double summerCharge(int quantity) {
        return quantity * _summerRate;
    }
    private double winterCharge(int quantity) {
        return quantity * _winterRate + _winterServiceCharge;
    }

### 9.2 Consolidate Conditional Expression
motivations:  
a sequence of conditional tests with the same result.

summary:  
Combine them into a single conditional expression and extract it.

example:  

	double disabilityAmount() {
        if (_seniority < 2) return 0;
        if (_monthsDisabled > 12) return 0;
        if (_isPartTime) return 0;
    }
↓

	double disabilityAmount() {
        if (isNotEligableForDisability()) return 0;
    }

### 9.3 Consolidate Duplicate Conditional Fragments
The same fragment of code is in all branches of a conditional expression.

example:  

	if (isSpecialDeal()) {
        total = price * 0.95;
        send();
    }
    else {
        total = price * 0.98;
        send();
    }
↓

	if (isSpecialDeal())
        total = price * 0.95;
    else
        total = price * 0.98;
    send();

### 9.4 Remove Control Flag
a variable that is acting as a control flag for a series of boolean expressions.

Use a **break or return** instead.

example:  

	void checkSecurity(String[] people) {
        boolean found = false;
        for (int i = 0; i < people.length; i++) {
            if (! found) {
                if (people[i].equals ("Don")){
                    sendAlert();
                    found = true;
                }
                if (people[i].equals ("John")){
                    sendAlert();
                    found = true;
                }
            }
        }
    }
↓

	void checkSecurity(String[] people) {
        for (int i = 0; i < people.length; i++) {
            if (people[i].equals ("Don")){
                sendAlert();
                break;
            }
            if (people[i].equals ("John")){
                sendAlert();
                break;
            }
        }
    }

### 9.5 Replace Nested Conditional with Guard Clauses
A method has conditional behavior that does not make clear the normal path of execution.

summary:  
Use guard clauses for all the special cases.

example:  

	double getPayAmount() {
        double result;
        if (_isDead) result = deadAmount();
        else {
            if (_isSeparated) result = separatedAmount();
            else {
                if (_isRetired) result = retiredAmount();
                else result = normalPayAmount();
            };
        }
        return result;
    };
↓

	double getPayAmount() {
        if (_isDead) return deadAmount();
        if (_isSeparated) return separatedAmount();
        if (_isRetired) return retiredAmount();
        return normalPayAmount();
    };

or, reversing the conditions:

	public double getAdjustedCapital() {
        double result = 0.0;
        if (_capital > 0.0) {
            if (_intRate > 0.0 && _duration > 0.0) {
                result = (_income / _duration) * ADJ_FACTOR;
            }
        }
        return result;
    }
↓

	public double getAdjustedCapital() {
        if (_capital <= 0.0) return 0.0;
        if (_intRate <= 0.0 || _duration <= 0.0) return 0.0;
        return (_income / _duration) * ADJ_FACTOR;
    }

### 9.6 Replace Conditional with Polymorphism
Move each leg of the conditional to an overriding method in a subclass. Make the original method abstract.


example:  

	class Employee{
        int payAmount() {
            switch (getType()) {
                case EmployeeType.ENGINEER:
                    return _monthlySalary;
                case EmployeeType.SALESMAN:
                    return _monthlySalary + _commission;
                case EmployeeType.MANAGER:
                    return _monthlySalary + _bonus;
                default:
                    throw new RuntimeException("Incorrect Employee");
            }
        }
	}
↓

	abstract class Employee{
        abstract int payAmount();
    }
    class Engineer extends Employee{
        int payAmount(){
            return _monthlySalary;
        }
    }
    class Salesman extends Employee{
        int payAmount(){
            return _monthlySalary + _commission;
        }
    }
    ...

### 9.7 Introduce Null Object
summary:  
have repeated checks for a null value. Replace the null value with a null object.  

(..........)

### 9.8 Introduce Assertion
A section of code assumes something about the state of the program.  

Make the assumption explicit with an assertion.

example:  

	double getExpenseLimit() {
        // should have either expense limit or a primary project
        return (_expenseLimit != NULL_EXPENSE) ?
            _expenseLimit:
            _primaryProject.getMemberExpenseLimit();
    }
↓

	double getExpenseLimit() {
        Assert.isTrue (_expenseLimit != NULL_EXPENSE || _primaryProject != null);
        return (_expenseLimit != NULL_EXPENSE) ?
            _expenseLimit:
            _primaryProject.getMemberExpenseLimit();
    }

# 10. Making Method Calls Simpler
### 10.1 Rename Method
The name of a method does not reveal its purpose.

Change the name of the method.

### 10.2 Add Parameter
A method needs more information from its caller.

Add a parameter for an object that can pass on this information.

the reverse of this is *Remove Parameter*

### 10.2 Remove Parameter
the reverse of *Add Parameter*

### 10.3 Separate Query from Modifier
