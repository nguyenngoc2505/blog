# Service Object là gì? Sử dụng nó như thế nào?
Thông thường khi bắt đầu làm một ứng dụng web với Ruby on Rails, với những yêu cầu đơn giản, việc thực thi business logic trong model có thể không gây ảnh hưởng gì lớn đối với hệ thống, cũng như việc đọc, hiểu logic của ứng dụng. Tuy nhiên khi mà yêu cầu tăng lên, các chức năng mới được thêm vào, để giữ cho controller đơn giản bạn phải chuyển việc thực thi các chức năng vào trong model điều đó làm cho model phình to ra, gây cản trở cho việc debug, test, hay đơn giản việc rất khó khăn cho việc đọc hiểu, bảo trì,hay truyền đạt lại logic cho những người sau. Để cải thiện tình trạng này ta có thể sử dụng `Service Object` như một lớp mới trong thiết kế của chúng ta. Vậy `Service Object` là gì? Sử dụng nó như thế nào? Bài viết sau sẽ đưa ra khái niệm khách quan về nó và một ví dụ nhỏ để ta có thể hình dung ra `Service Object`.

## Service Object là gì?
`Service Object` là nơi thực thi tương tác của người dùng với ứng dụng. Nó chứa tất cả business logic của một model => ta cũng có thể coi nó như là core của ứng dụng. Để hiểu hơn về nó ta xét ví dụ cụ thể như sau:
- Xét model `User`, mỗi `user` có thuộc tính tuổi(age) dựa vào đó ta có thể biết được họ đang trong độ tuổi gì?(đã về hưu - retired?, hay là đang trong độ tuổi lao động - working?)

Theo ví dụ trên ta có được mối liên hệ giữa service object với mô hình MVC như sau:
- **Model(User):** mỗi một user sẽ có thuộc tính tuổi(age) là một số nguyên và lớn hơn 15 tuổi.
- **View:** hiện thị đúng thông tin phù hợp với từng độ tuổi.
- **Controller:** nhận kết quả từ service object truyền xuống view.
- **Service Object:** xác định độ tuổi của user, xem trả về giá trị boolean cho biết người đó có đang nghỉ hưu? hay đang trong độ tuổi lao động hay không? hay đơn giản xác định xem thuộc độ tuổi nào?

## Sử dụng Service Object
Tất cả các service đều có điểm chung giống nhau là cách sử dụng:
- Chấp nhận dữ liệu đầu vào
- Xử lý dữ liệu
- Trả về kết quả

Chúng ta hãy cùng xem chi tiết từng giai đoạn cụ thể.

### Khởi tạo object
Bởi vì một service thực thi tương tác của người dùng với ứng dụng nên nó thường được khởi tạo như một object, chứa toàn bộ thông tin mà user nhập vào

### Dữ liệu nhập vào (input).
Service Object chấp nhận tất cả các giá trị nhập vào của user, có thể là một object, single value, hash value, array...

Theo ví dụ trên input ở đây có thể là object user hay tuổi của người đó, tùy thuộc vào mục đích sử dụng và các tham số cần lấy ra.
- Ở ví dụ dưới đây chúng ta sẽ dùng thử đầu vào là 1 object.
```ruby
class User::CheckAge
  attr_reader :user
  PERIOD_RETIRED = 55

  #Khởi tạo mới một service object với tham số truyền vào là một object user
  def initialize user
    @user = user
  end

  #Kiểm tra user đó có thuộc độ tuổi nghỉ hưu hay không?
  def retired?
    age >= PERIOD_RETIRED
  end

  #Kiểm tra user đó thuộc độ tuổi làm việc hay không?
  def working?
    !retired?
  end

  #Trả về loại độ tuổi của user đó.
  def type_of_age
    retired? ? "Retired" : "Working"
  end

  private
  def age
    user.age
  end

  def gender
    #... có thể trả về giới tính của user đó
  end
end
```
### Xử lý logic & Kết quả trả về
- Với dữ liệu ban đầu được truyền vào, Service Object sẽ sử dụng các dữ liệu đó để thực hiện các logic phức tạp như: tạo, cập nhật, xóa một, nhiều record, hay gửi thông báo, email, hoặc thực hiện các chức năng mới ...Ở ví dụ trên, việc xử lý logic đơn giản là kiểm tra xem user thuộc độ tuổi nào. Việc sử dụng serviec object sẽ giúp "tách biệt" các logic phức tạp(business logic) và các logic liên quan đến framework như model hay controller.
- Sau một quá trình xử lý logic thì cần phải có một kết quả trả về theo mong đợi của người dùng. Với Service Object thì kết quả trả về là bất cứ loại giá trị nào mà mình mong muốn, nó có thể là 1 ActiveRecord Object hay là Boolean hay Single value... Xét ví dụ kiểm tra độ tuổi của User ở trên ta sẽ có một số giá trị trả về như sau:

 - Trả về giá trị boolean
  ```ruby
  User::CheckAge.new(User.first).retired?
  => true or false
  ```
 - Trả về single value
  ```ruby
  User::CheckAge.new(User.first).type_of_age
  => "Retired" or "Working"
  ```

## Testing Services
Khi service object chứa business logic của hệ thống thì việc test service trở nên quan trọng. Tuy nhiên chính việc tách riêng business logic ra thành service riêng làm cho việc test trở nên dễ dàng hơn rất nhiều.
Dưới đây là ví dụ rspec cho model `User` và service trên:

```ruby
describe CheckAge do

  describe "#retired?" do
    context 'is retired' do
      let(:user) {FactoryGirl.create :user, age: 55}
      subject {User::CheckAge.new(user).retired?}
      it {is_expected.to be true}
    end
    context 'is not retired' do
      let(:user) {FactoryGirl.create :user, age: 30}
      subject {User::CheckAge.new(user).retired?}
      it {is_expected.to be false}
    end
  end

  describe "#working?" do
    context 'is working' do
      let(:user) {FactoryGirl.create :user, age: 25}
      subject {User::CheckAge.new(user).working?}
      it {is_expected.to be true}
    end
    context 'is not working' do
      let(:user) {FactoryGirl.create :user, age: 55}
      subject {User::CheckAge.new(user).working?}
      it {is_expected.to be false}
    end
  end

  describe "#type_of_age" do
    subject {User::CheckAge.new(user).type_of_age}
    context "when age is greater than retired period" do
      let(:user) {FactoryGirl.create :user, age: 56}
      it {is_expected.to eq("Retired")}
    end
    context "when age is less than retired period" do
      let(:user) {FactoryGirl.create :user, age: 25}
      it {is_expected.to eq("Working")}
    end
  end

end
```
## Kết luận
Trên đây giới thiệu Service Object là gì và một ví dụ tổng quan nhỏ để bạn có thể hình dung rõ ràng hơn về Service. Qua đó các bạn cũng thấy được khi nào thì nên dùng Service, những ưu điểm, lợi thế của nó mang lại.

- Ưu điểm:
  - Tách riêng được logic phức tạp ra khỏi controller, model giữ cho nó đơn giản nhất.
  - Việc logic được tách riêng ra giúp cho việc test, debug dễ dàng hơn, hơn nữa khi mà yêu cầu bị thay đổi sẽ dễ dàng sửa đổi trong từng service mà không bị ảnh hưởng nhiều bởi các hàm liên quan.

- Có đạt được những ưu điểm mà service object mang lại hay không còn phụ thuộc nhiều vào thiết kế của bạn. Nếu như có một thiết kế tồi, không quản lý được các service thực thi được chức năng gì hay để các chức năng chồng chéo lên nhau => bị nhầm lẫn, khó quản lý.

## Giải nghĩa
- Business Logic: là phần code đại diện cho các yêu cầu của doanh nghiệp, hay đối tác, khách hàng, nó xác định xem dữ liệu cần được tiếp nhận, xử lý và trình bày như thế nào.
