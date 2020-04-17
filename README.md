# UI
## 1. Tạo root mới (tham khảo HomeRoot.jsx)
- Kế thừa từ BaseHomeRoot
- Overwrite proto baseHomeRootDidMount(callback). Hàm này được sử dụng để bắn api lấy danh sách menu về, hàm callback bắt buộc phải được gọi (truyền vào tham số callback của các hàm update state).
- Overwrite proto getRouteList. Hàm này cần return về list object chứa thông tin các url tồn tại trong root này.
- Bắt buộc window.renderPage phải render withStyles(x)(Component), trong đó x là object merge giữa baseHomeRootStyles và các style nội bộ.
## 2. Component chính cho trang: cần có layout để kế thừa
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 2.1 PageLayout: sử dụng cho các trang mà người dùng sử dụng chính (tham khảo HomePage.jsx)
- Overwrite this.title: Tiêu đề trang
- Overwrite this.actionButtons: biến này để được gán bằng 1 mảng, mỗi phần tử của mảng phải là kết quả của hàm this.getActionButtonObj(label, icon class, onClickFunc)
- Overwrite proto renderHeader: hàm này render header

## 3. Validate wrapper
Được sử dụng khi muốn validate theo actionId truyền vào, trong component này sẽ xử lý, gọi api validate,...
Cách dùng 
```jsx
    <ValidateWrapper
      onClick={() => { console.log(1); }}
      alwaysDisplay={true}
      actionId={EActionPoint.Test}>
      <Button color="success" onHover={ () => { alert(1); } }>
        Validate
      </Button>
    </ValidateWrapper>
```
- Theo như code trên thì khi click Buton, validate wrapper sẽ làm 1 số thao tác kiểm tra quyền theo prop actionId, nếu thỏa thì prop onClick mới chạy. 
- Prop alwaysDisplay nếu bằng false thì nếu validate bị false thì khối bên trong (trường hợp này là Button) sẽ không hiện lên.
- Nếu các function muốn thực hiện trong khối mà không cần validate thì truyền trực tiếp vào component bên trong (Prop onHover trong ví dụ)

## 4.Phím tắt
- Để đăng kí được phím tắt thì component đó bắt buộc phải kế thừa từ EHealthBaseConsumer
- Để đăng kí thì gọi this.registerHotKey(hotKey, function)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - hotKey: lấy từ enum EHotKey
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - function: hàm invoke khi nhấn đúng hot key
- Thông thường hàm this.registerHotKey được gọi ở componentDidMount của Component (hoặc có thể đăng kí bất cứ lúc nào);
- Mặc định thì các hotKey sẽ được hủy đăng kí tự động khi componentWillUnmount, tuy nhiên vẫn có thể chủ động hủy đăng kí bằng cách gọi this.unregisterHotKey(hotKey, function). Chú ý, function phải cùng pointer với function ở hàm đăng kí.
