# UI
## 1. Tạo root mới (tham khảo HomeRoot.jsx)
- Kế thừa từ BaseHomeRoot
- Overwrite proto baseHomeRootDidMount(callback). Hàm này được sử dụng để bắn api lấy danh sách menu về, hàm callback bắt buộc phải được gọi (truyền vào tham số callback của các hàm update state).
- Overwrite proto getRouteList. Hàm này cần return về list object chứa thông tin các url tồn tại trong root này.
- Bắt buộc window.renderPage phải render withStyles(x)(Component), trong đó x là object merge giữa baseHomeRootStyles và các style nội bộ.

<a name="page-layout"/>
## 2. Component chính cho trang: cần có layout để kế thừa
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 2.1 PageLayout: sử dụng cho các trang mà người dùng sử dụng chính (tham khảo HomePage.jsx)
- Overwrite `this.title`: Tiêu đề trang
- Overwrite `this.actionButtons`: biến này để được gán bằng 1 mảng, mỗi phần tử của mảng phải là kết quả của hàm `this.getActionButtonObj(label, icon class, onClickFunc)`
- Overwrite proto `renderHeader`: hàm này render header
- Không được kế thừa `componentDidMount`, override hàm `loadData` để get dữ liệu, với parameter của callback function, function này cần được gọi khi đã load data xong, xem VD dưới
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
- các component trong modal phải kế thừa `ModalLayout` và implement hàm `modalBody` để render nội dung modal
- các modal kế thừa `ModalLayout` tắt modal đó bằng cách gọi hàm `this.closeThisModal();`
- Modal thường được sử dụng để user thực hiện một hành động, thường là nhập một form input gì đó vì vậy việc user nhấn nút X hay đại loại là thoát modal thì phải kiếm tra xem user đã có input gì chưa bằng cách lưu lại data ban đầu, và so sánh với data lúc thoát modal, nếu muốn có behaviour này thì cần:
    - Override hàm `dataToCompare`, hàm này return về data mà user sẽ thay đổi
    - Gọi hàm `this.setInitDataToCompare(x)` ở didMount của modal với `x` là data mà user sẽ thay đổi
    
    VD:
    ```jsx
    export default class CreatePatientModal extends ModalLayout {
        dataToCompare() {
            return this.props.data;
        }
        componentDidMount() {
            this.setInitDataToCompare(this.props.data);
        }
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
## 8. Select / Select Async
### EhealthSelect - sử dụng để select khi đã có options
#### Props
Name | Type | Default | Description
:--- | :--- | :--- | :---
`renderOption` | function | `option => option.label;` | Render label
`variant` | "simple" or "complex" | `"complex"` | Đơn giản hay màu mè
`onChange` | function | | callback function khi change, parameter là list khi `multiple == true`, object khi `multiple == false`
`search` | boolean | `false` | `true` nếu cần filter options
`fullWidth` | boolean | `false` | 
`multiple` | boolean | `false` |
`label` | string | `"Select"` | Label (placeholder)
`options` | array | | list of object of {label, value}
`value` | array or object | | array nếu multiple, ngược lại
`leftIcon` | node | `null` | icon bênh trái `label`
`noOptionText` | string | `"No options"` | text display khi không có options
```jsx
import EHealthSelect from "~/components/ui-kit/Select/EHealthSelect";
<EHealthSelect
	label="Select multiple"
	options={options}
	value={options.filter(i => myValues.includes(i.value))}
	onChange={selectedOptions => {
		this.overwiteList(myValues, selectedOptions.map(i => i.value));
	}}
	multiple
/>
```

### EHealthSelectAsync - sử dụng để select query từ back-end
#### Props
Bao gồm props của `EHealthSelect` và:
Name | Type | Default | Description
:--- | :--- | :--- | :---
`getValue` | function | | lấy value cho select dựa vào options, xem VD dưới
`getInit` | boolean | `false` | `true` nếu cần lấy sẵn options 
`apiUrl`* | string | | api query options từ back-end, được nối vào với searchValue, xem VD dưới
`extractOptionsFromApi`* | function | | Trích xuất options từ `ack`, xem VD dưới

```jsx
import EHealthSelectAsync from "~/components/ui-kit/Select/EHealthSelectAsync";
<EHealthSelectAsync
    label="Select multiple"
    apiUrl="/api/Doctor/GetAllergyOptions?searchValue="
    extractOptionsFromApi={ack => ack.data.items}
    getValue={options => options.find(i => myValue == i.value)}
    onChange={selectedOption => {
        this.updateObject(data, {myValue: selectedOption.value});
    }}
/>
```

