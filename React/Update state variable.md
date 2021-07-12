# Hướng dẫn update state variable khi sử dụng useState hook

Bài viết này chỉ tập trung vào cách để update lại giá trị của state variable. Để tìm hiểu cơ bản về useState hook hoặc các hook khác khi là việc với function component các bạn có thể search trên google.

    Có hai điểm cần nhớ khi update một object đã được state:
    1. React sẽ không render lại view khi bạn sử dụng cùng một object để update state.
    2. Setter được return bởi useState sẽ không merge object giống như setState trong class component.


Sau đây chúng ta sẽ sử dụng các ví dụ để làm rõ hai điểm trên.

`Điểm số 1`. Nếu bạn sử dụng cùng một value để update lại state variable thì React sẽ không render lại view. Vì React sẽ sử dụng Object.is để so sánh hai biến với nhau, nếu cùng là một biến thì không có sự thay đổi.

Ví dụ bên dưới là một sai lầm mà chúng ta thường gặp đặc biệt là với các bạn mới.

    const Message = () => {
        const [messageObj, setMessage] = useState({ message: '' });
        return (
            <div>
                <input
                    type="text"
                    value={messageObj.message}
                    placeholder="Enter a message"
                    onChange={e => {
                        messageObj.message = e.target.value;
                        setMessage(messageObj); // view sẽ không cập nhật giá trị như mong muốn
                    }}
                />
                <p>
                    <strong>{messageObj.message}</strong>
                </p>
            </div>
        );
    };


Thay vì sử dụng luôn variable cũ để truyền cho setter. Chúng ta phải tạo một variable mới.

    onChange={e => {
        const newMessageObj = { message: e.target.value };
        setMessage(newMessageObj); // Now it works
    }}

`Điểm số 2`. Setter được return bởi useState sẽ không merge object giống như setState trong class component.

Cùng với ví dụ trên, nếu bây giờ có thêm một id property.

    const Message = () => {
        const [messageObj, setMessage] = useState({ message: '', id: 1 });

        return (
            <div>
                <input
                    type="text"
                    value={messageObj.message}
                    placeholder="Enter a message"
                    onChange={e => {
                    const newMessageObj = { message: e.target.value };
                    setMessage(newMessageObj); 
                    }}
                />
                <p>
                    <strong>{messageObj.id} : {messageObj.message}</strong>
                </p>
            </div>
        );
    };

Với ví dụ này, chúng ta tạo một newMessageObj chỉ có message property. Sau khi sự kiện onChange được xử lý. giá trị ban đầu đã mất thuộc tính id.

    { message: '', id: 1 } -> { message: 'message entered' }

Như vậy React đã không tự merge hai object lại với nhau.

### `Làm sao để sử dụng useState tối ưu nhất?`

Hãy xem ví dụ dưới đây:

    onChange={e => {
        const val = e.target.value;
        setMessage(prevState => {
            return { ...prevState, message: val }
        });
    }}

Chúng ta sẽ truyền một function argument cho setter để merge giá trị cũ và mới theo cú pháp

    { ...prevState, message: val }

Ngoài ra, đối với array, chúng ta cũng sử dụng cách tương tự như với object.

    const MessageList = () => {
        const [message, setMessage] = useState("");
        const [messageList, setMessageList] = useState([]);

        return (
            <div>
                <input
                    type="text"
                    value={message}
                    placeholder="Enter a message"
                    onChange={e => {
                    setMessage(e.target.value);
                    }}
                />
                <input
                    type="button"
                    value="Add"
                    onClick={e => {
                    setMessageList([
                        ...messageList,
                        {
                        // Use the current size as ID (needed to iterate the list later)
                        id: messageList.length + 1,
                        message: message
                        }
                    ]);
                    setMessage(""); // Clear the text box
                    }}
                />
                <ul>
                    {messageList.map(m => (
                    <li key={m.id}>{m.message}</li>
                    ))}
                </ul>
            </div>
        );
    };


Tham khảo và trích dẫn.
https://blog.logrocket.com/a-guide-to-usestate-in-react-ecb9952e406c/