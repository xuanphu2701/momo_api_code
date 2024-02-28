1.	Tổng quan
1.1	Luồng xử lý
 
Hình 1: Luồng xử lí API

1.2	Đặc tả API
Key	Value
Content-Type	application/json; charset=UTF-8
Method	POST
Domain	Production: https://payment.momo.vn
Sandbox: https://test-payment.momo.vn


1.3	Thông tin đơn hàng
Attribute	Type	Required	Description
partnerCode	String	x	Thông tin tích hợp

subPartnerCode	String		Định danh duy nhất của tài khoản M4B của bạn
storeName	String		Tên đối tác
storeId	String		Mã cửa hàng
requestId	String(50)	x	Định danh duy nhất cho mỗi yêu cầu
Đối tác sử dụng requestId để xử lý idempotency

amount	Long	x	Số tiền cần thanh toán
Nhỏ Nhất: 1.000 VND
Tối đa: 50.000.000 VND
Tiền tệ: VND
Kiểu dữ liệu: Long
orderId	String	x	Mã đơn hàng của đối tác

orderInfo	String	x	Thông tin đơn hàng
orderGroupId	Long		orderGroupId được MoMo cung cấp để phân nhóm đơn hàng cho các hoạt động vận hành sau này. Vui lòng liên hệ với MoMo để biết chi tiết cách sử dụng
redirectUrl	String	x	Một URL của đối tác. URL này được sử dụng để chuyển trang (redirect) từ MoMo về trang mua hàng của đối tác sau khi khách hàng thanh toán.Hỗ trợ: AppLink and WebLink
ipnUrl	String	x	API của đối tác. Được MoMo sử dụng để gửi kết quả thanh toán theo phương thức IPN (server-to-server)
requestType	String	x	captureWallet
extraData	String	x	Giá trị mặc định là rỗng""
Encode base64 theo định dạng Json: {"key": "value"}
Ví dụ với dữ liệu: {"username": "momo"} thì data extraData: eyJ1c2VybmFtZSI6ICJtb21vIn0="
items	List		Danh sách các sản phẩm hiển thị trên trang thanh toán.
Tối đa: 50 loại sản phẩm
deliveryInfo	Object		Thông tin giao hàng của đơn hàng
userInfo	Object		Thông tin người dùng
referenceId	String		Mã tham chiếu phụ của đối tác
Ví dụ dùng trong các trường hợp như mã khách hàng, mã hộ gia đình, mã hóa đơn, mã thuê bao v.v
autoCapture	Boolean		Nếu giá trị false, giao dịch sẽ không tự động capture. Mặc định là true
lang	String	x	Ngôn ngữ của message được trả về (vi hoặc en)
signature	String	x	Chữ ký để xác nhận giao dịch. Sử dụng thuật toán Hmac_SHA256 với data theo định dạng được sort từ a-z: accessKey=$accessKey&amount=$amount&extraData=$extraData&ipnUrl=$ipnUrl&orderId=$orderId&orderInfo=$orderInfo&partnerCode=$partnerCode&redirectUrl=$redirectUrl&requestId=$requestId&requestType=$requestType
Điều hướng thông tin (redirectUrl)
-	WebLink: Link để mở website.
-	AppLink: Link để mở mobile application.
1.4	Tạo signature
mac = HMAC(hmac_algorihtm, secretKey, hmac_input)
Trong đó:
•	hmac_algorithm: Là phương thức bảo mật do ứng dụng đăng ký với Momo, mặc định là HmacSHA256
•	secretKey: Là secret key do Momo cung cấp.
•	hmac_input: điền theo format sau: accessKey=$accessKey&amount=$amount&extraData=$extraData&ipnUrl=$ipnUrl&orderId=$orderId&orderInfo=$orderInfo&partnerCode=$partnerCode&redirectUrl=$redirectUrl&requestId=$requestId&requestType=$requestType

2.	Hướng dẫn sử dụng Momo - API
2.1	Tạo tham số API sử dụng code
Dưới đây là đoạn code dùng để generate và tạo request đến Momo (thay đổi các biến accessKey, secretKey và partnerCode):
//change this var 
var partnerCode = 'MOMOYQBV20240223_TEST';
var accessKey = 'rLngVVmkvIs0DbBt';
var secretKey = 'hbUnrAnMUaAkQSlSHKB7iPSsYHD8fGi6';

var orderInfo = 'pay with MoMo';
var redirectUrl = 'https://webhook.site/b3088a6a-2d17-4f8d-a383-71389a6c600b';
var ipnUrl = 'https://webhook.site/b3088a6a-2d17-4f8d-a383-71389a6c600b';
var requestType = "captureWallet";
var amount = '50000';
var orderId = partnerCode + new Date().getTime();
var requestId = orderId;
var extraData ='';
var orderGroupId ='';
var autoCapture =true;
var lang = 'vi';

//before sign HMAC SHA256 with format
//accessKey=$accessKey&amount=$amount&extraData=$extraData&ipnUrl=$ipnUrl&orderId=$orderId&orderInfo=$orderInfo&partnerCode=$partnerCode&redirectUrl=$redirectUrl&requestId=$requestId&requestType=$requestType
var rawSignature = "accessKey=" + accessKey + "&amount=" + amount + "&extraData=" + extraData + "&ipnUrl=" + ipnUrl + "&orderId=" + orderId + "&orderInfo=" + orderInfo + "&partnerCode=" + partnerCode + "&redirectUrl=" + redirectUrl + "&requestId=" + requestId + "&requestType=" + requestType;
//puts raw signature
console.log("--------------------RAW SIGNATURE----------------")
console.log(rawSignature)
//signature
const crypto = require('crypto');
var signature = crypto.createHmac('sha256', secretKey)
    .update(rawSignature)
    .digest('hex');
console.log("--------------------SIGNATURE----------------")
console.log(signature)

//json object send to MoMo endpoint
const requestBody = JSON.stringify({
    partnerCode : partnerCode,
    partnerName : "Test",
    storeId : "MomoTestStore",
    requestId : requestId,
    amount : amount,
    orderId : orderId,
    orderInfo : orderInfo,
    redirectUrl : redirectUrl,
    ipnUrl : ipnUrl,
    lang : lang,
    requestType: requestType,
    autoCapture: autoCapture,
    extraData : extraData,
    orderGroupId: orderGroupId,
    signature : signature
});
//Create the HTTPS objects
const https = require('https');
const options = {
    hostname: 'test-payment.momo.vn',
    port: 443,
    path: '/v2/gateway/api/create',
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Content-Length': Buffer.byteLength(requestBody)
    }
}
//Send the request and get the response
const req = https.request(options, res => {
    console.log(`Status: ${res.statusCode}`);
    console.log(`Headers: ${JSON.stringify(res.headers)}`);
    res.setEncoding('utf8');
    res.on('data', (body) => {
        console.log('Body: ');
        console.log(body);
        console.log('resultCode: ');
        console.log(JSON.parse(body).resultCode);
    });
    res.on('end', () => {
        console.log('No more data in response.');
    });
})

req.on('error', (e) => {
    console.log(`problem with request: ${e.message}`);
});
// write data to request body
console.log("Sending....")
req.write(requestBody);
req.end();

Sau khi gửi request thành công, ta nhận được dữ liệu trả về như sau:
 
Hình 2: Dữ liệu trả về thành công
2.2	Sử dụng Postman
Người dùng truy cập ứng dụng Postman và tạo các request với body như bên dưới: 
 
Hình 3: Tạo request và folder trong Postman
Sau khi hoàn thành việc nhập dữ liệu, người dùng nhấn vào nút [Send] để nhận tham số API trả về:
 
Hình 4: Tham số API trả về khi giao dịch thành công
2.3	Test thanh toán
Sau khi nhận được đường link trong trường payUrl, ta vào đường link sẽ được chuyển tới cổng thanh toán momo: 
 
Hình 5: Cổng thanh toán bằng mã QR
Sau đó sử dụng phần mềm Momo Test để tiến hành thanh toán. Sau khi quét mã QR ta sẽ được chuyển về phần thanh toán trên app Momo:
  
Hình 6: Tiến hành thanh toán trên app Momo Test
3.	Tham khảo
- Thanh toán momo API: https://developers.momo.vn/v3/vi/docs/payment/api/wallet/onetime/
- Code tham khảo sử dụng để request API: 
