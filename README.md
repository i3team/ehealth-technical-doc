# UI
## 1. Tạo root mới (tham khảo HomeRoot.jsx)
- Kế thừa từ BaseHomeRoot
- Overwrite proto baseHomeRootDidMount(callback). Hàm này được sử dụng để bắn api lấy danh sách menu về, hàm callback bắt buộc phải được gọi (truyền vào tham số callback của các hàm update state).
- Overwrite proto getRouteList. Hàm này cần return về list object chứa thông tin các url tồn tại trong root này.
- Bắt buộc window.renderPage phải render withStyles(x)(Component), trong đó x là object merge giữa baseHomeRootStyles và các style nội bộ.

<a name="page-layout"/>

## 2. Page Layout
Sử dụng cho các trang mà người dùng sử dụng chính (tham khảo HomePage.jsx)
- Overwrite `this.title`: Tiêu đề trang
- Overwrite `this.actionButtons`: biến này để được gán bằng 1 mảng, mỗi phần tử của mảng phải là kết quả của hàm `this.getActionButtonObj(label, icon class, onClickFunc)`
- Overwrite proto `renderHeader`: hàm này render header
- Không được ghi đè `componentDidMount`, ghi đè hàm `loadData` để get dữ liệu, với parameter của callback function, function này cần được gọi khi đã load data xong, xem VD dưới
```jsx
loadData(finishLoading) {
	this.ajaxGet({
		url: `/apiurl`,
		success: ack => {
			this.updateObject(this.props.data, ack.data, finishLoading); // gọi callback
		}
	})
}
```

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

## 5. Modal
- `openModal` có parameter thứ 2 là modal type (xem enum `EModalType`)
```jsx
let mdf = () => ({
    title: "Modal title",
    body: (
        <Modal props />
    )
})
this.openModal(mdf, ModalType.Right)
```
[Để dễ hình dung, xem demo design về modal tại đây](https://xd.adobe.com/view/31ba350f-eda0-49c2-5928-79cef4d01083-478b/screen/984dbfbe-3dcd-4d52-ae14-982365e2e2e1/Right-Modal-)

- các component trong modal phải kế thừa `ModalLayout`
    - Implement hàm `modalBody` để render nội dung modal
    - Override hàm `leftFooter` hoặc/và `rightFooter` để render footer theo từng phía (xem demo bên trên)
- các modal kế thừa `ModalLayout` tắt modal đó bằng cách gọi hàm `this.closeThisModal();`
- Modal thường được sử dụng để user thực hiện một hành động, thường là nhập một form input gì đó vì vậy việc user nhấn nút X hay đại loại là thoát modal thì phải kiếm tra xem user đã có input gì chưa bằng cách lưu lại data ban đầu, và so sánh với data lúc thoát modal, nếu muốn có behaviour này thì cần:
    - Override hàm `dataToCompare`, hàm này return về data mà user sẽ thay đổi
    - Gọi hàm `this.setInitDataToCompare(x)` ở didMount của modal với `x` là data mà user sẽ thay đổi
    - Override hàm `closeModalIfClickAway() : boolean`, return `true` nếu muốn tắt modal khi click ra ngoài modal, và ngược lại
    - Override hàm `onClose()` nếu `closeModalIfClickAway` return `true`, hàm được gọi sau khi tắt modal
    - Override hàm `hasDefaultPadding() : boolean`, return `true` nếu có default padding (giá trị này trước đó được truyền dưới dạng parameter ở hàm `openModal`)
    
    VD:
    ```jsx
    export default class CreatePatientModal extends ModalLayout {
        dataToCompare() {
            return this.props.data;
        }
        componentDidMount() {
            this.setInitDataToCompare(this.props.data);
        }
        // override hàm rightFooter để render nút Lưu ở bên phải
        rightFooter(){
            return (
                <div>
                    <button>Lưu</button>
                </div>
            )
        }
    	closeModalIfClickAway() {
		return true;
	}
    	hasDefaultPadding() {
		return true;
    	}
	onClose(){
		console.log('modal closed, do what u want');
	}
	
        // override hàm rightFooter để render nút Đóng (modal) ở bên trái
        leftFooter(){
            return (
                <div>
                   Đóng
                </div>
            )
        }
        // implement body
        modalBody() {
            const { data, ...others } = this.props;
            return (
                <div>
                    <input value={data.name} onChange={this._onChangeName} />
                </div>
            )
        }
    }
    ```
## 6. Display date
```jsx
import helper from '~/general/helper';

let date = helper.displayDate(new Date())
let dateTime = helper.displayDateTime(new Date())
```

# Các component UI-kit
## 1. ButtonGroup
Sử dụng để render nhiều button nằm cạnh nhau trên một hàng
## 2. ButtonWithIcon
Render 1 button có 1 icon và text nằm cạnh bên
## 3. DevidedComponents
Render nhiều components với thanh dọc chia giữa các components với nhau
## 4. EHealthTable
Bảng dữ liệu có chức năng render thêm checkbox ở mỗi dòng
## 5. EHealthSelectionTable
Bảng dữ liệu extends từ EHealthTable, mỗi item trong datalist truyền vô phải có cấu trúc của 1 Selectable (class C#).
Trong class này đã handle sẵn các event onChange của checkbox, nếu có ít nhất 1 item được check thì sẽ hiện lên drawer ở bottom
## 6. LineShadow
Đường ngang thể hiện box shadow
## 7. VerticalDevider
Đường dọc để ngăn cách các components với nhau


## Form
Để sử dụng được chức năng Enter để submit 1 form thì cần bao các component (input, select,...) bằng thẻ <form></form>
````jsx
<form onSubmit={this._submit}>
	<I3TextField ... />
	<Button type="submit">Tìm</Button> (hoặc <Button onClick={this._submit}>Tìm</Button>)
</form>

_submit = (e)=> {
	e.preventDefault(); //ngăn load lại trang
	//your code here
}
````

# Backend
## 1. Phân trang
Model sử dụng để phân trang là Pagination (Pagination.cs)
```csharh
    public class Pagination
    {
        public int PageIndex { get; set; }
        public int PageSize { get; set; }
    }

    public class Pagination<T>: Pagination
    {
        public List<T> DataList { get; set; }
        public int TotalPage { get; set; }
        public int TotalItem { get; set; }
    }
```
## 2. DataWrapper
Model sử dụng để lấy kèm options cho việc select
```csharp
public class DataWrapper<T>
{
    public T Data { get; set; }
    public Dictionary<int, List<Selectable>> OptionDict { get; set; }
}

public enum EOptionKey : int
{
    BloodType = 1,
    BloodRhType = 2,
}
```
Ngoài ra cũng có thể có như cầu lấy kèm cả 1 model filter hay search model gì đó
```csharp
public class DataWrapper<TData, TFilter> : DataWrapper<TData>
{
    public TFilter Filter { get; set; }
}
```


Khi get dữ liệu cần lấy thêm options để select, nên sẽ xảy ra việc tạo một model mới để wrap lại, đây là lúc model này "ra tay"
```csharp
public Acknowledgement<DataWrapper<MedicalRecord>> GetMedicalRecord()
{
    var ack = new Acknowledgement<DataWrapper<MedicalRecord>>();
    ack.Data = new DataWrapper<MedicalRecord>();
            
    // Lấy data bình thường
    ack.Data.Data = new MedicalRecord();

    // Lấy các list options khác nhau
    ack.Data.OptionDict = new Dictionary<int, List<Selectable>>();
            
    // VD: Lấy options cho status của bệnh án
    var statusOptions = EnumHelper.GetSelectableOptions<EMedicalRecordStatus>();
    ack.Data.OptionDict.Add((int)EOptionKey.Status, statusOptions);
    // Nếu có thêm options khác thì add thêm vào dict
    ack.IsSuccess = true;
    return ack;
}

// Lấy kèm filter
public Acknowledgement<DataWrapper<List<MedicalRecord>, RecordPageFilterModel>> GetModelRecordList()
{
}
```
Sử dụng ở front-end React
```jsx
// file enum.js
const EOptionKey = {
    BloodType: 1,
    BloodRhType: 2,
    Status: 3,
}

// sử dụng
this.ajaxGet({
	url: '~/GetMedicalRecord',
	success: ack => {
		let statusOptions = ack.data.optionDict[EOptionKey.Status];
	}
})
```

## 3. Thay đổi tên trường virtual edmx
Xem và note trong file EHealth.POCO/CustomFields.txt
