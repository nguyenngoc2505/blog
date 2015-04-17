---
Title: .nil? .empty? .blank? và .present?
Author: Nguyễn Đức Giang
---

# .nil? .empty? .blank? và .present?

`.blank?` và `.present?` là những hàm mình thường xuyên sử dụng từ khi mới bắt đầu làm quen với Rails. Sự tiện dụng là lí do những hàm này được dùng thường xuyên, nhưng mình bắt đầu từ việc *những người khác cũng dùng như thế*. Sau một thời gian mình *biết thêm* `.nil?` và `empty?` và trở nên bối rối (thời điểm đó mình còn chưa biết xem doc và không có khái niệm tách biệt về Ruby và Rails): những hàm này khác nhau thế nào? Khi nào thì nên dùng cái nào?

Theo Google thì dường như có rất nhiều người thắc mắc như vậy, và cũng nhiều bài viết về vấn đề này. Mình xin góp thêm một bài bằng tiếng Việt dựa trên những gì mình tìm hiểu được.

Trước hết:

- `.nil?` và `.empty?` là hàm của Ruby
- `.blank` và `.present?` là các hàm được thêm trong Rails

# `.nil?`

Theo [Ruby doc](http://ruby-doc.org/core-2.2.0/Object.html#method-i-nil-3F) thì:

- `.nil?` là một hàm của Object, nên tất cả các object kế thừa từ Object mặc định đều có hàm `nil?`
- chỉ có `nil` object trả về `true` khi gọi `nil?`

```rb
nil.nil?
# => true
"".nil?
# => false
4.nil?
# => false
Object.new.nil?
# => false
BasicObject.new.nil?
# NoMethodError: undefined method `nil?' for #<BasicObject:0x0000000b794b30> from (pry):72:in `<main>'
```

# `.empty?`

`.empty?` là function có sẵn của [String](http://ruby-doc.org/core-2.1.0/String.html#method-i-empty-3F), [Array](http://ruby-doc.org/core-2.2.0/Array.html#method-i-empty-3F), [Hash](http://ruby-doc.org/core-2.2.0/Hash.html#method-i-empty-3F)

Hiểu một cách đơn giản thì `.empty?` sẽ trả về `true` nếu:

- string.length == 0
- array.length == 0
- hash.length == 0

Như vậy,

```rb
"".empty?
# => true
[].empty?
# => true
{}.empty?
# => true
" ".empty?
# => false
[nil].empty?
# => false
```

# `.blank?`

`.blank?` là một hàm rất thú vị của Rails, cũng như được sử dụng rất thường xuyên (cùng với `.present?`)

Theo như mô tả trong [source code](https://github.com/rails/rails/blob/5e51bdda59c9ba8e5faf86294e3e431bd45f1830/activesupport/lib/active_support/core_ext/object/blank.rb): một object được coi là `blank` nếu như nó `false`, `empty` hoặc là 1 chuỗi chỉ gồm các khoảng trắng.

Điều này cho phép ta dự đoán:

```rb
# những object dưới đây khi gọi `.blank?` sẽ trả về true vì chúng `empty`
[].blank? # => true
{}.blank? # => true
"".blank? # => true
# những object dưới đây khi gọi `.blank?` sẽ trả về true vì chúng `false`
false.blank? # => true
nil.blank? # => true
# những object dưới đây khi gọi `.blank?` sẽ trả về true vì chúng là chuỗi chỉ có khoảng trắng
"   ".blank? # => true
"\s".blank? # => true
# như vậy, những object dưới đây nên trả về false khi gọi `.blank?`
true.blank? # => false
5.blank? # => false
```

Thử các lệnh trên trong `rails console` xác nhận các suy đoán trên là đúng.

Hãy cùng xem Rails định nghĩa `.blank?` ra sao (sao chép từ mã nguồn của Rails, đã lược bỏ chú thích):

```rb
class Object
  def blank?
    respond_to?(:empty?) ? !!empty? : !self
  end
end

class NilClass
  def blank?
    true
  end
end

class FalseClass
  def blank?
    true
  end
end

class TrueClass
  def blank?
    false
  end
end

class Array
  alias_method :blank?, :empty?
end

class Hash
  alias_method :blank?, :empty?
end

class String
  BLANK_RE = /\A[[:space:]]*\z/
  def blank?
    BLANK_RE === self
  end
end

class Numeric
  def blank?
    false
  end
end
```

Quên đi định nghĩa của `.blank?` trong lớp `Object`, xem phần còn lại ta sẽ thấy với `Array` và `Hash` thì `.blank?` chẳng qua là alias method của `.empty?`. Còn trong các lớp `NilClass`, `FalseClass` thì `.blank?` trực tiếp trả về `false`, trong các lớp `TrueClass`, `Numeric` thì `.blank?` trực tiếp trả về true.

Với lớp `String`, do có nhiều loại kí tự khoảng trắng (như space, tab) nên Rails sử dụng regex `/[[:space:]]/` từ Ruby (xem [Regexp](http://ruby-doc.org/core-2.1.1/Regexp.html)). Điều này có nghĩa rằng tất cả các loại kí tự được Ruby xem là khoảng trắng sẽ được dùng cho việc xác định một chuỗi có gồm toàn khoảng trắng hay không, thể hiện qua ví dụ dưới đây:

```rb
# chuỗi này chỉ bao gồm toàn các kí tự khoảng trắng
"\t\n\r\s   \u00a0".blank?
# => true
```

Vậy, như phần trên đã nói chỉ có `Array`, `Hash` và `String` có `.empty?`, trong khi `.blank?` trong các lớp trên đều đã bị khai báo đè lên trên khai báo của `Object`. Vậy tại sao `.blank?` trong Object vẫn được khai báo như sau:

```rb
class Object
  def blank?
    respond_to?(:empty?) ? !!empty? : !self
  end
end
```

Việc tìm hiểu ý nghĩa của đoạn mã này phụ thuộc vào bạn, mình sẽ tạm dừng câu chuyện về `.blank?` tại đây.

# `.present?`

Nếu 1 object không `blank`, vậy nó `present`.

```rb
  # An object is present if it's not blank.
  #
  # @return [true, false]
  def present?
    !blank?
  end
```

Như vậy không có gì nhiều để nói về `.present?` nữa, tuy nhiên có một hàm có liên quan mật thiết đến `.present?` mà chúng ta cũng rất hay sử dụng, đó là `.presence`

```rb
def presence
  self if present?
end
```

Với định nghĩa này ta có thể suy ra rằng `.presence` sẽ trả về object gọi nó nếu object đó `present`, còn không sẽ trả về `nil`.

Một ví dụ cho `.presence`:

```rb
s = "  " || "default"
# => "  "
s = "  ".presence || "default"
# => "default"
```

# Kết luận

Việc hiểu bản chất của các hàm `.nil?`, `empty?`, `.blank?` và `present?` cho phép  bạn thấy được sự khác nhau, cũng như hoàn cảnh mà bạn nên áp dụng từng hàm, giúp cho các đoạn mã của bạn rõ ràng hơn.
