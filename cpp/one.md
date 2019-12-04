# 第一节

1. **C++实现split函数**
   ```c++
   std::vector<std::string> split_v1(std::string & str) {
     std::istringstream iss(str);
     return std::vector<std::string>(std::istream_iterator<std::string>{iss}, std::istream_iterator<std::string>());
   }
   /////////////////////////////////////////////////////////////////////////////
   template<char delimiter>
   class  MyString : public std::string {};
   template<char delimiter>
   std::istream& operator>>(std::istream& is, MyString<delimiter>& output) {
     std::getline(is, output, delimiter);
     return is;
   }
   template<char delimiter>
   std::vector<std::string> split_v2(std::string& str) {
     std::istringstream iss(str);
     return std::vector<std::string>(std::istream_iterator<MyString<delimiter>>{iss}, std::istream_iterator<MyString<delimiter>>());
   }
   /////////////////////////////////////////////////////////////////////////////
   std::vector<std::string> split_v3(const std::string& s, char delimiter) {
     std::vector<std::string> tokens;
     std::string token;
     std::istringstream token_stream(s);
     while (std::getline(token_stream, token, delimiter)) {
       tokens.push_back(token);
     }
     return tokens;
   }
   ```
   * `split_v1`方法简单，只使用了STL的东西，由于istringstream的特性，这里只能按空格切割字符串。
   * `split_v2`可以通过`template`在编译器指定分隔符。
   * `split_v3`在运行时决定分隔符。
