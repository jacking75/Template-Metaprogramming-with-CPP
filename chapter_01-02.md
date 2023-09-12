# chapter 1
실행은 온라인 컴파일러를 사용한다.  https://wandbox.org/   
    
<br>  

## 템플릿을 사용하지 않을 때    
  
```
#include <iostream>

namespace n101
{
   // 여러 버전의 max 함수를 만들어야 한다
   int max(int const a, int const b)
   {
      return a > b ? a : b;
   }

   double max(double const a, double const b)
   {
      return a > b ? a : b;
   }


   using swap_fn = void(*)(void*, int const, int const);
   using compare_fn = bool(*)(void*, int const, int const);

   // 타입을 추상화하기 위해 void* 사용
   int partition(void* arr, int const low, int const high,
      compare_fn fcomp, swap_fn fswap)
   {
      int i = low - 1;

      for (int j = low; j <= high - 1; j++)
      {
         if (fcomp(arr, j, high))
         {
            i++;
            fswap(arr, i, j);
         }
      }

      fswap(arr, i + 1, high);

      return i + 1;
   }

   void quicksort(void* arr, int const low, int const high,
      compare_fn fcomp, swap_fn fswap)
   {
      if (low < high)
      {
         int const pi = partition(arr, low, high, fcomp, fswap);
         quicksort(arr, low, pi - 1, fcomp, fswap);
         quicksort(arr, pi + 1, high, fcomp, fswap);
      }
   }

   void swap_int(void* arr, int const i, int const j)
   {
      int* iarr = (int*)arr;
      int t = iarr[i];
      iarr[i] = iarr[j];
      iarr[j] = t;
   }

   bool less_int(void* arr, int const i, int const j)
   {
      int* iarr = (int*)arr;
      return iarr[i] <= iarr[j];
   }


   // int 타입만 저장할 수 있는 vector
   struct int_vector
   {
      int_vector();

      size_t size() const;
      size_t capacity() const;
      bool empty() const;

      void clear();
      void resize(size_t const size);

      void push_back(int value);
      void pop_back();

      int at(size_t const index) const;
      int operator[](size_t const index) const;
   private:
      int* data_;
      size_t size_;
      size_t capacity_;
   };

   constexpr char NewLine = '\n';
   constexpr wchar_t NewLineW = L'\n';
   constexpr char8_t NewLineU8 = u8'\n';
   constexpr char16_t NewLineU16 = u'\n';
   constexpr char32_t NewLineU32 = U'\n';
}
```
   
main.cpp  
```
int main()
{
   using namespace n101;

   int arr[] = { 13, 1, 8, 3, 5, 2, 1 };
   int n = sizeof(arr) / sizeof(arr[0]);
   quicksort(arr, 0, n - 1, less_int, swap_int);   
}
```  
  
<br>  
  

## 템플릿 사용  
  
```
namespace n102
{
   // 1개 버전의 함수만 만들면 된다
   template <typename T>
   T max(T const a, T const b)
   {
      return a > b ? a : b;
   }

   struct foo {};

   template <typename T>
   void swap(T* a, T* b)
   {
      T t = *a;
      *a = *b;
      *b = t;
   }

   template <typename T>
   int partition(T arr[], int const low, int const high)
   {
      T pivot = arr[high];
      int i = (low - 1);

      for (int j = low; j <= high - 1; j++)
      {
         if (arr[j] < pivot)
         {
            i++;
            swap(&arr[i], &arr[j]);
         }
      }

      swap(&arr[i + 1], &arr[high]);

      return i + 1;
   }

   template <typename T>
   void quicksort(T arr[], int const low, int const high)
   {
      if (low < high)
      {
         int const pi = partition(arr, low, high);
         quicksort(arr, low, pi - 1);
         quicksort(arr, pi + 1, high);
      }
   }

   template <typename T>
   struct vector
   {
      vector();

      size_t size() const;
      size_t capacity() const;
      bool empty() const;

      void clear();
      void resize(size_t const size);

      void push_back(T value);
      void pop_back();

      T at(size_t const index) const;
      T operator[](size_t const index) const;
   private:
      T* data_;
      size_t size_;
      size_t capacity_;
   };

   template<typename T>
   constexpr T NewLine = T('\n');
}
```  
  
main.cpp    
```  
int main()
{
   {
      using namespace n102;

      max(1, 2);        // OK, compares ints
      max(1.0, 2.0);    // OK, compares doubles

      //foo f1, f2;
      //max(f1, f2);      // Error, operator> not overloaded for foo

      max<int>(1, 2);
      max<double>(1.0, 2.0);
      //max<foo>(f1, f2);
   }

   {
      using namespace n102;

      int arr[] = { 13, 1, 8, 3, 5, 2, 1 };
      int n = sizeof(arr) / sizeof(arr[0]);
      quicksort(arr, 0, n - 1);
   }

   {
      using namespace n102;

      std::wstring test = L"demo";
      test += NewLine<wchar_t>;
      std::wcout << test;
   }
}
```  
  
<br>  
<br>  
  
  
# chapter 2
실행은 온라인 컴파일러를 사용한다.  https://wandbox.org/   
    
<br>  

## 라이브러리  
템플릿 타입을 멤버를 가지는 단수낳 `wrapper` 구조체  
  
wrapper.h  
```
#pragma once 

namespace ext
{
   template <typename T>
   struct wrapper
   {
      T data;
   }; 

   extern template wrapper<int>;

   void f();
   void g();
}
```    
  
source1.cpp  
```
#include "wrapper.h"
#include <iostream>

namespace ext
{
   template wrapper<int>;

   void f()
   {
      ext::wrapper<int> a{ 42 };

      std::cout << a.data << '\n';
   }
}
```   
  
source2.cpp  
```
#include "wrapper.h"
#include <iostream>

namespace ext
{
   void g()
   {
      wrapper<int> a{ 100 };

      std::cout << a.data << '\n';
   }
}
```  
  

### 2-1
단순한 템플릿 함수  
    
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n201
{
   template <typename T>
   T add(T const a, T const b)
   {
      return a + b;
   }

   class foo
   {
      int value;
   public:
      explicit foo(int const i) :value(i)
      {
      }

      explicit operator int() const { return value; }
   };

   foo operator+(foo const a, foo const b)
   {
      return foo((int)a + (int)b);
   }

   template <typename Input, typename Predicate>
   int count_if(Input start, Input end, Predicate p)
   {
      int total = 0;
      for (Input i = start; i != end; i++)
      {
         if (p(*i))
            total++;
      }
      return total;
   }
}


int main()
{
   {
      using namespace n201;

      auto a1 = add(42, 21);
      auto a2 = add<int>(42, 21);
      auto a3 = add<>(42, 21);

      auto b = add<short>(42, 21);

      //auto d1 = add(41.0, 21); // error
      auto d2 = add<double>(41.0, 21);

      auto f = add(foo(42), foo(41));

      int arr[]{ 1,1,2,3,5,8,11 };
      int odds = count_if(std::begin(arr), std::end(arr),
         [](int const n) { return n % 2 == 1; });
      std::cout << odds << '\n';
   }
}
```  

## 202    
단순한 템플릿 함수  
    
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n202
{
   template <typename T>
   class wrapper
   {
   public:
      wrapper(T const v):value(v)
      { }

      T const& get() const { return value; }
   private:
      T value;
   };
}

int main()
{
   {
      using namespace n202;

      wrapper a(42);           // wraps an int
      wrapper<int> b(42);      // wraps an int
      wrapper<short> c(42);    // wraps a short
      wrapper<double> d(42.0); // wraps a double
      wrapper e("42");         // wraps a char const *
   }
}
```
  
## 203
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n203
{
   template <typename T>
   class wrapper;

   void use_wrapper(wrapper<int>* ptr);
}

namespace n203
{
   template <typename T>
   class wrapper
   {
   public:
      wrapper(T const v) :value(v)
      {
      }

      T const& get() const { return value; }
   private:
      T value;
   };

   void use_wrapper(wrapper<int>* ptr)
   {
      std::cout << ptr->get() << '\n';
   }
}

int main()
{
  {
      using namespace n203;

      wrapper<int> a(42);          // wraps an int
      use_wrapper(&a);
   }
}
```
    

## 204
템플릿 선언이 클래스 생성자에 하는 경우  
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

namespace n204
{
   template <typename T>
   class composition
   {
   public:
      T add(T const a, T const b)
      {
         return a + b;
      }
   };
}


int main()
{
   {
      using namespace n204;

      composition<int> c;
      c.add(41, 21);  // int 타입의 데이터만 인자로 가능하다
   }
}
```
    

## 205
템플릿 선언이 클래스 생성자가 아닌 멤버 함수에 하는 경우  

```  
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

namespace n205
{
   class composition
   {
   public:
      template <typename T>
      T add(T const a, T const b)
      {
         return a + b;
      }
   };
}

int main()
{
   {
      using namespace n205;

      composition c;
      c.add<int>(41, 21);
   }
}
```   
    
## 206
템플릿 선언이 클래스 생성자와 멤버함수에 각각 있는 경우  
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

namespace n206
{
   template <typename T>
   class wrapper
   {
   public:
      wrapper(T const v) :value(v)
      {
      }

      T const& get() const { return value; }

      template <typename U>
      U as() const
      {
         return static_cast<U>(value);
      }
   private:
      T value;
   };
}

int main()
{
   {
      using namespace n206;

      wrapper<double> a(42.0);
      auto d = a.get();       // double
      auto n = a.as<int>();   // int
   }
}
```
    
   
### 209  
크기(사이즈)를 가지는 클래스 템플릿  
  
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>


namespace n209
{
   template <typename T, size_t S>
   class buffer
   {
      T data_[S];
   public:
      constexpr T const * data() const { return data_; }

      constexpr T& operator[](size_t const index)
      {
         return data_[index];
      }

      constexpr T const & operator[](size_t const index) const
      {
         return data_[index];
      }
   };

   template <typename T, size_t S>
   buffer<T, S> make_buffer()
   {
      return {};
   }
}

int main()
{
   {
      using namespace n209;

      buffer<int, 10> b1;
      b1[0] = 42;
      std::cout << b1[0] << '\n';

      auto b2 = make_buffer<int, 10>();      
   }

   {
      using namespace n209;

      buffer<int, 10> b1;
      buffer<int, 2*5> b2;

      std::cout << std::is_same_v<buffer<int, 10>, buffer<int, 2*5>> << '\n';
      std::cout << std::is_same_v<decltype(b1), decltype(b2)> << '\n';

      static_assert(std::is_same_v<decltype(b1), decltype(b2)>);

      buffer<int, 3*5> b3;
      static_assert(!std::is_same_v<decltype(b1), decltype(b3)>);
   }
}
```  
결과  
<pre>  
42
1
1
</pre>  
  
  
### 210
Command 객체의 멤버 함수를 사용하는 템플릿  

```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>


namespace n210
{
   struct device
   {
      virtual void output() = 0;
      virtual ~device() {}
   };

   template <typename Command, void (Command::*action)()>
   struct smart_device : device
   {
      smart_device(Command* command) : cmd(command) {}

      void output() override
      {
         (cmd->*action)();
      }
   private:
      Command* cmd;
   };

   struct hello_command
   {
      void say_hello_in_english()
      {
         std::cout << "Hello, world!\n";
      }

      void say_hello_in_spanish()
      {
         std::cout << "Hola mundo!\n";
      }
   };
}

int main()
{
   {
      using namespace n210;

      hello_command cmd;

      auto w1 = std::make_unique<smart_device<hello_command, &hello_command::say_hello_in_english>>(&cmd);
      w1->output();

      auto w2 = std::make_unique<smart_device<hello_command, &hello_command::say_hello_in_spanish>>(&cmd);
      w2->output();
   }
}
```    
결과   
<pre>
Hello, world!
Hola mundo!
</pre>  
  

### 211
210과 비슷  
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>


namespace n211
{
   struct device
   {
      virtual void output() = 0;
      virtual ~device() {}
   };

   template <typename Command, void (Command::*action)()>
   struct smart_device : device
   {
      smart_device(Command& command) : cmd(command) {}

      void output() override
      {
         (cmd.*action)();
      }
   private:
      Command& cmd;
   };

   struct hello_command
   {
      void say_hello_in_english()
      {
         std::cout << "Hello, world!\n";
      }

      void say_hello_in_spanish()
      {
         std::cout << "Hola mundo!\n";
      }
   };
}

int main()
{
   {
      using namespace n211;

      hello_command cmd;

      auto w1 = std::make_unique<smart_device<hello_command, &hello_command::say_hello_in_english>>(cmd);
      w1->output();

      auto w2 = std::make_unique<smart_device<hello_command, &hello_command::say_hello_in_spanish>>(cmd);
      w2->output();
   }
}
```
  
### 212 
210을 다르게 구현. 인자로 일반 함수를 받는 경우    
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

namespace n212
{
   struct device
   {
      virtual void output() = 0;
      virtual ~device() {}
   };

   template <void (*action)()>
   struct smart_device : device
   {
      void output() override
      {
         (*action)();
      }
   };

   void say_hello_in_english()
   {
      std::cout << "Hello, world!\n";
   }

   void say_hello_in_spanish()
   {
      std::cout << "Hola mundo!\n";
   }
}

int main()
{
   {
      using namespace n212;

      auto w1 = std::make_unique<smart_device<&say_hello_in_english>>();
      w1->output();

      auto w2 = std::make_unique<smart_device<&say_hello_in_spanish>>();
      w2->output();

      static_assert(!std::is_same_v<decltype(w1), decltype(w2)>);
   }

   {
      using namespace n212;

      std::unique_ptr<device> w1 = std::make_unique<smart_device<&say_hello_in_english>>();
      w1->output();

      std::unique_ptr<device> w2 = std::make_unique<smart_device<&say_hello_in_spanish>>();
      w2->output();

      static_assert(std::is_same_v<decltype(w1), decltype(w2)>);
   }
}
```
  

### 213
템플릿 인자로 리터럴 문자열을 사용한 경우  
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

  
namespace n213
{
   template <auto x>
   struct foo
   {
   };
}

int main()
{
   {
      using namespace n213;
      [[maybe_unused]] foo<42> f1;    // foo<int>
      [[maybe_unused]] foo<42.0> f2;  // foo<double>
      //foo<"42"> f3;  // error: '"42"' is not a valid template argument for type 'const char*' because string literals can never be used in this
   }
}
```
   

### 214 
213의 에러를 해결한 버전   
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>
  
  
namespace n214
{
   template<size_t N>
   struct string_literal
   {
      constexpr string_literal(const char(&str)[N])
      {
         std::copy_n(str, N, value);
      }

      char value[N];
   };

   template <string_literal x>
   struct foo
   {
   };
}

int main()
{
   {
      using namespace n214;
      [[maybe_unused]] foo<"42"> f;
   }
}
```
  
### 215
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

  
namespace n215
{
   template<auto... x>
   struct foo
   { /* ... */ };
}

int main()
{
   {
      using namespace n215;
      [[maybe_unused]] foo<42, 42.0, false, 'x'> f; // 만약 'x'가 아닌 "x"라면 에러
   }
}
```
  

### 216
`template<typename>` 사용 예. 템플릿의 템플릿 객체를 멤버로 가지는 경우 사용.  
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n216
{
   template <typename T>
   class simple_wrapper
   {
   public:
      T value;
   };

   template <typename T>
   class fancy_wrapper
   {
   public:
      fancy_wrapper(T const v) :value(v)
      {
      }

      T const& get() const { return value; }

      template <typename U>
      U as() const
      {
         return static_cast<U>(value);
      }
   private:
      T value;
   };

   template <typename T, typename U, template<typename> typename W = fancy_wrapper>
   class wrapping_pair
   {
   public:
      wrapping_pair(T const a, U const b) :
         item1(a), item2(b)
      {
      }

      W<T> item1;
      W<U> item2;
   };
}

int main()
{
   {
      using namespace n216;
      wrapping_pair<int, double> p1(42, 42.0);
      std::cout << p1.item1.get() << ' '
                << p1.item2.get() << '\n';

      wrapping_pair<int, double, simple_wrapper> p2(42, 42.0);
      std::cout << p2.item1.value << ' '
                << p2.item2.value << '\n';
   }
}
```


### 219
default 템플릿 파라미터  
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>


namespace n219
{
   template <typename T, typename U = double>
   struct foo;

   template <typename T = int, typename U>
   struct foo;

   template <typename T, typename U>
   struct foo
   {
      T a;
      U b;
   };
}

int main()
{
   {
      using namespace n219;
      foo f{ 42, 42.0 };
   }
}
```
    

### 221
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

namespace n221
{
   template <typename T>
   struct foo
   {
   protected:
      using value_type = T;
   };

   template <typename T, typename U = typename T::value_type>
   struct bar
   {
      using value_type = U;
   };
}

int main()
{
   {
      using namespace n221;
      bar<foo<int>> x;
   }
}
```
  
      
### 223
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

namespace n223
{
   template <typename T>
   struct foo
   {
      void f() {}
      void g() {}
   };
}

int main()
{
   {
      using namespace n223;

      [[maybe_unused]] foo<int>* p;
      [[maybe_unused]] foo<int> x;
      [[maybe_unused]] foo<double>* q;
   }

   {
      using namespace n223;

      [[maybe_unused]] foo<int>* p;
      foo<int> x;
      foo<double>* q = nullptr;

      x.f();
      q->g(); // 멤버 변수를 접근하지 않으므로 에러 발생하지 않음
   }
}
```
  
### 225
멤버 변수가 `static`  
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

namespace n225
{
   template <typename T>
   struct foo
   {
      static T data;
   };

   template <typename T> T foo<T>::data = 0;
}

int main()
{
   {
      using namespace n225;

      foo<int> a;
      foo<double> b;
      foo<double> c;

      std::cout << a.data << '\n'; // 0
      std::cout << b.data << '\n'; // 0
      std::cout << c.data << '\n'; // 0

      b.data = 42;
      std::cout << b.data << '\n'; // 42
      std::cout << c.data << '\n'; // 42
   }   
}
```  
  

### 228
`constexpr`, 템플릿 특수화    
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>


namespace n228
{
   template <typename T>
   struct is_floating_point
   {
      constexpr static bool value = false;
   };

   template <>
   struct is_floating_point<float>
   {
      constexpr static bool value = true;
   };

   template <>
   struct is_floating_point<double>
   {
      constexpr static bool value = true;
   };

   template <>
   struct is_floating_point<long double>
   {
      constexpr static bool value = true;
   };

   template <typename T>
   inline constexpr bool is_floating_point_v = is_floating_point<T>::value;
}

int main()
{
   {
      using namespace n228;

      std::cout << is_floating_point<int>::value << '\n';
      std::cout << is_floating_point<float>::value << '\n';
      std::cout << is_floating_point<double>::value << '\n';
      std::cout << is_floating_point<long double>::value << '\n';
      std::cout << is_floating_point<std::string>::value << '\n';
   }

   {
      using namespace n228;

      std::cout << is_floating_point_v<int> << '\n';
      std::cout << is_floating_point_v<float> << '\n';
      std::cout << is_floating_point_v<double> << '\n';
      std::cout << is_floating_point_v<long double> << '\n';
      std::cout << is_floating_point_v<std::string> << '\n';
   }
}
```  
결과  
<pre>
0
1
1
1
0
0
1
1
1
0
</pre>
   
   
### 229
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n229
{
   template <typename T>
   struct is_floating_point;

   template <>
   struct is_floating_point<float>
   {
      constexpr static bool value = true;
   };

   template <typename T>
   struct is_floating_point
   {
      constexpr static bool value = false;
   };
}

int main()
{
   {
      using namespace n229;

      std::cout << is_floating_point<int>::value << '\n';
      std::cout << is_floating_point<float>::value << '\n';
   }
}
```  
  
###
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n230
{
   template <typename>
   struct foo {};    // primary template

   template <>
   struct foo<int>;  // explicit specialization declaration
}

int main()
{
   {
      using namespace n230;

      [[maybe_unused]] foo<double> a; // OK
      [[maybe_unused]] foo<int>* b;   // OK
      //foo<int> c;    // error, foo<int> incomplete type
   }
}
```
  
### 231
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n231
{
   template <typename T>
   struct foo {};

   template <typename T>
   void func(foo<T>)
   {
      std::cout << "primary template\n";
   }

   template<>
   void func(foo<int>)
   {
      std::cout << "int specialization\n";
   }
}

int main()
{
   {
      using namespace n231;

      func(foo<int>{});
      func(foo<double>{});
   }
}
```
  
### 232 
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n232
{
   template <typename T>
   void func(T a)
   {
      std::cout << "primary template\n";
   }

   template <>
   void func(int a /*= 0*/) // error: default argument
   {
      std::cout << "int specialization\n";
   }
}

int main()
{
   {
      using namespace n232;

      func(42.0);
      func(42);
   }
}
```  
  
### 233
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n233
{
   template <typename T>
   struct foo
   {
      static T value;
   };

   template <typename T> T foo<T>::value = 0;

   template <> int foo<int>::value = 42;
}

int main()
{
   {
      using namespace n233;

      foo<double> a, b;
      std::cout << a.value << '\n';
      std::cout << b.value << '\n';

      foo<int> c;
      std::cout << c.value << '\n';

      a.value = 100;
      std::cout << a.value << '\n';
      std::cout << b.value << '\n';
      std::cout << c.value << '\n';
   }
}
```
     
### 234 
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n234
{
   template <typename T, typename U>
   void func(T a, U b)
   {
      std::cout << "primary template\n";
   }

   template <>
   void func(int a, int b)
   {
      std::cout << "int-int specialization\n";
   }

   template <>
   void func(int a, double b)
   {
      std::cout << "int-double specialization\n";
   }
}

int main()
{
   {
      using namespace n234;

      func(1, 2);
      func(1, 2.0);
      func(1.0, 2.0);
   }
}
```
  
### 235
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n235
{
   template <typename T, int S>
   struct collection
   {
      void operator()()
      { 
         std::cout << "primary template\n";
      }
   };

   template <typename T>
   struct collection<T, 10>
   {
      void operator()()
      {
         std::cout << "partial specialization <T, 10>\n";
      }
   };

   template <int S>
   struct collection<int, S>
   { 
      void operator()()
      {
         std::cout << "partial specialization <int, S>\n";
      }
   };

   template <typename T, int S>
   struct collection<T*, S>
   { 
      void operator()()
      {
         std::cout << "partial specialization <T*, S>\n";
      }
   };
}

int main()
{
   {
      using namespace n235;

      collection<char, 42>{}();  // primary template
      collection<int, 42>{}();   // partial specialization <int, S>
      collection<char, 10>{}();  // partial specialization <T, 10>
      collection<int*, 20>{}();  // partial specialization <T*, S>

      //collection<int, 10>{}();      // error: collection<T,10> or collection<int,S>
      //collection<char*, 10>{}();    // error: collection<T,10> or collection<T*,S>
   }
}
```
  
### 237
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n237
{
   template <typename T, size_t S>
   std::ostream& pretty_print(std::ostream& os, std::array<T, S> const& arr)
   {
      os << '[';
      if (S > 0)
      {
         size_t i = 0;
         for (; i < S - 1; ++i)
            os << arr[i] << ',';
         os << arr[S - 1];
      }
      os << ']';

      return os;
   }
}

int main()
{
   {
      using namespace n237;

      std::array<int, 9> arr {1, 1, 2, 3, 5, 8, 13, 21};
      pretty_print(std::cout, arr);

      std::array<char, 9> str;
      std::strcpy(str.data(), "template");
      pretty_print(std::cout, str);
   }
}
```
  
### 238
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n238
{
   template <size_t S>
   std::ostream& pretty_print(std::ostream& os, std::array<char, S> const& arr)
   {
      os << '[';
      for (auto const& e : arr)
         os << e;
      os << ']';

      return os;
   }
}

int main()
{
   {
      using namespace n238;

      std::array<char, 9> str;
      std::strcpy(str.data(), "template");
      pretty_print(std::cout, str);
   }
}
```
  
### 239
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n239
{
   constexpr double PI = 3.1415926535897932385L;

   template <typename T>
   T sphere_volume(T const r)
   {
      return static_cast<T>(4 * PI * r * r * r / 3);
   }
}  

int main()
{
   {
      using namespace n239;

      float v1 = sphere_volume(42.0f);
      double v2 = sphere_volume(42.0);
   }
}
```
  
### 240
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"


namespace n240
{
   template <typename T>
   struct PI
   {
      static const T value;
   };

   template <typename T> 
   const T PI<T>::value = T(3.1415926535897932385L);

   template <typename T>
   T sphere_volume(T const r)
   {
      return 4 * PI<T>::value * r * r * r / 3;
   }
}

int main()
{
   {
      using namespace n240;

      float v1 = sphere_volume(42.0f);
      double v2 = sphere_volume(42.0);
   }
}
```
   
### 241
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n241
{
   template<class T>
   constexpr T PI = T(3.1415926535897932385L);

   template <typename T>
   T sphere_volume(T const r)
   {
      return 4 * PI<T> * r * r * r / 3;
   }
}

int main()
{
   {
      using namespace n241;

      float v1 = sphere_volume(42.0f);
      double v2 = sphere_volume(42.0);
   }
}
```
  
### 242
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n242
{
   struct math_constants
   {
      template<class T>
      static constexpr T PI = T(3.1415926535897932385L);
   };

   template <typename T>
   T sphere_volume(T const r)
   {
      return 4 * math_constants::PI<T> *r * r * r / 3;
   }
}

int main()
{
   {
      using namespace n242;

      float v1 = sphere_volume(42.0f);
      double v2 = sphere_volume(42.0);
   }
}
```
  
### 244
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n244
{
   template<typename T> 
   constexpr T SEPARATOR = '\n';

   template<> 
   constexpr wchar_t SEPARATOR<wchar_t> = L'\n';

   template <typename T>
   std::basic_ostream<T>& show_parts(std::basic_ostream<T>& s, 
                                     std::basic_string_view<T> const& str)
   {
      using size_type = typename std::basic_string_view<T>::size_type;
      size_type start = 0;
      size_type end;
      do
      {
         end = str.find(SEPARATOR<T>, start);
         s << '[' << str.substr(start, end - start) << ']' << SEPARATOR<T>;
         start = end+1;
      } while (end != std::string::npos);

      return s;
   }
}

int main()
{
   {
      using namespace n244;
      show_parts<char>(std::cout, "one\ntwo\nthree");
      show_parts<wchar_t>(std::wcout, L"one line");
   }

   {
      typedef int index_t;
      typedef std::vector<std::pair<int, std::string>> NameValueList;
      typedef int (*fn_ptr)(int, char);

   }

   {
      using index_t = int;
      using NameValueList = std::vector<std::pair<int, std::string>>;
      using fn_ptr = int(int, char);
   }
}
```
   
### 247
```
#include <iostream>
#include <array>
#include <string>
#include <string_view>
#include <vector>
#include <map>
#include <type_traits>
#include <array>
#include <numeric>
#include <algorithm>
#include <functional>

#include "wrapper.h"

namespace n247
{
   template <typename T>
   using customer_addresses_t = std::map<int, std::vector<T>>;

   struct delivery_address_t {};
   struct invoice_address_t {};

   using customer_delivery_addresses_t = customer_addresses_t<delivery_address_t>;
   using customer_invoice_addresses_t = customer_addresses_t<invoice_address_t>;
}

namespace n247
{
   template <typename T, size_t S>
   struct list
   {
      using type = std::vector<T>;
   };

   template <typename T>
   struct list<T, 1>
   {
      using type = T;
   };

   template <typename T, size_t S>
   using list_t = typename list<T, S>::type;
}

int main()
{
   {
      using namespace n247;
      static_assert(std::is_same_v<list_t<int, 1>, int>);
      static_assert(std::is_same_v<list_t<int, 2>, std::vector<int>>);
   }

   {
      int arr[] = { 1,6,3,8,4,2,9 };
      std::sort(
         std::begin(arr), std::end(arr),
         [](int const a, int const b) {return a > b; });

      int pivot = 5;
      auto count = std::count_if(
         std::begin(arr), std::end(arr),
         [pivot](int const a) {return a > pivot; });

      std::cout << count << '\n';
   }

   {
      auto l1 = [](int a) {return a + a; };           // C++11, regular lambda
      auto l2 = [](auto a) {return a + a; };          // C++14, generic lambda
      auto l3 = []<typename T>(T a) { return a + a; };// C++20, template lambda

      auto v1 = l1(42);	                  // OK
      auto v2 = l1(42.0);	               // warning
      //auto v3 = l1(std::string{ "42" });   // error

      auto v5 = l2(42);                    // OK
      auto v6 = l2(42.0);                  // OK
      auto v7 = l2(std::string{"42"});     // OK

      auto v8 = l3(42);                    // OK
      auto v9 = l3(42.0);                  // OK
      auto v10 = l3(std::string{ "42" });   // OK
   }

   {
      auto l1 = [](int a, int b) {return a + b; };
      auto l2 = [](auto a, auto b) {return a + b; };
      auto l3 = []<typename T, typename U>(T a, U b) { return a + b; };
      auto l4 = [](auto a, decltype(a) b) {return a + b; };

      auto v1 = l1(42, 1);	                  // OK
      auto v2 = l1(42.0, 1.0);	               // warning
      //auto v3 = l1(std::string{ "42" }, '1'); // error

      auto v4 = l2(42, 1);                    // OK
      auto v5 = l2(42.0, 1);                  // OK
      auto v6 = l2(std::string{ "42" }, '1'); // OK
      auto v7 = l2(std::string{ "42" }, std::string{ "1" }); // OK

      auto v8 = l3(42, 1);                    // OK
      auto v9 = l3(42.0, 1);                  // OK
      auto v10 = l3(std::string{ "42" }, '1'); // OK
      auto v11 = l3(std::string{ "42" }, std::string{ "42" }); // OK

      auto v12 = l4(42.0, 1); // OK
      auto v13 = l4(42, 1.0); // warning
      //auto v14 = l4(std::string{ "42" }, '1'); // error
   }

   {
      auto l = []<typename T, size_t N>(std::array<T, N> const& arr) { return std::accumulate(arr.begin(), arr.end(), static_cast<T>(0)); };

      //auto v1 = l(1); // error
      auto v2 = l(std::array<int, 3>{1, 2, 3});
   }

   {
      auto l = []<typename T>(T a, T b) { return a + b; };

      auto v1 = l(42, 1);        // OK
      //auto v2 = l(42.0, 1);    // error

      auto v4 = l(42.0, 1.0);    // OK
      //auto v5 = l(42, false);    // error

      auto v6 = l(std::string{ "42" }, std::string{ "1" }); // OK
      //auto v6 = l(std::string{ "42" }, '1');                // error
   }

   {
      std::function<int(int)> factorial;
      factorial = [&factorial](int const n) {
         if (n < 2) return 1;
         else return n * factorial(n - 1);
      };

      std::cout << factorial(5) << '\n';
   }

   {
      auto factorial = [](auto f, int const n) {
         if (n < 2) return 1;
         else return n * f(f, n - 1);
      };

      std::cout << factorial(factorial, 5) << '\n';
   }
}
```  



 