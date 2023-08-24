# EVENT GROUP

## Tổng quan

> Cũng như `Semaphore`, `Queue`, `Mutex` thì `Even Group` có thể đưa một task vào trạng thái **Block** cũng như là **Running**. Tuy nhiên, khác ở chỗ là task bị **Block** có thể phải chờ nhiều sự kiện.

## Tại sao lại dùng Event Group

> Các ứng dụng về lập trình nhúng đòi hỏi phải người lập trình phải tối ưu về bộ nhớ. Băng cách sử dụng `Event Group` ta có thể tránh được việc phải sử dụng nhiều `Binary Semaphore`.

## Event Bits (Event Flags)

> `Event Bits` được sử dụng để chỉ định sự kiện đó có hoạt động hay không. `Event Bits` có tên gọi khác là `Event Flags`. Nó mang hai giá trị **_0_** và **_1_**.
> Ví dụ về cách ứng dụng trong lập trình:
> Define một bit (hoặc flag) có nghĩa "một tin nhắn được nhận và sẳn sàng để xử lý" khi nó được đặt thành 1 và "không có tin nhằn nào chờ được xử lý" khi nó được đặt thành 0.

## Event Group

> `Event Group` là một tập hợp các `Event Bits`.
> Kiểu dữ liệu `EventBits_t` được dùng để chứa các `Event Bits` trong một `Event Group`. Còn các `Event Group` có kiểu dữ liệu `EventGroupHandle_t`.
> Số `Event Bit` trong một `Event Group` tùy thuộc vào cấu hình của người lập trình ở **configUSE_16_BIT_TICKS** hoặc **configTICK_TYPE_WIDTH_IN_BITS**:
>
> -   Số bit (hoặc cờ) được triển khai trong một nhóm sự kiện là 8 nếu **configUSE_16_BIT_TICKS** được đặt thành 1 hoặc 24 nếu **configUSE_16_BIT_TICKS** được đặt thành 0.
> -   Số bit (hoặc cờ) được triển khai trong một nhóm sự kiện là 8 nếu **configTICK_TYPE_WIDTH_IN_BITS** được đặt thành **TICK_TYPE_WIDTH_16_BITS** hoặc 24 nếu **configTICK_TYPE_WIDTH_IN_BITS** được đặt thành **TICK_TYPE_WIDTH_32_BITS** hoặc 56 nếu **configTICK_TYPE_WIDTH_IN_BITS** được đặt thành **TICK*TYPE* WIDTH_64_BITS**.
>     ![Alt text](image/img_0.jpg)

## Những khó khăn khi sử dụng Event Group

> Có hai khó khăn chính trong việc sử dụng `Event Group` trong RTOS:
>
> -   Tránh tạo ra **race condition** trong ứng dụng của người dùng. Việc sử dụng `Event Group` tạo ra **race condition** nếu:
>     -   Không rõ ai sẽ chịu trách nhiệm xóa từng bit (hoặc cờ riêng lẻ).
>     -   Không rõ khi nào sẽ xóa bit.
>     -   Không rõ trạng thái của một bit do có một task khác hoặc một ngắt khác đã thay đổi bit đó trong quá trình chạy chương trình.
> -   Tránh **Non-Determinism**:
>     **Non-Determinism** có nghĩa là không xác định được có bao nhiêu task đang bị **Block** do đó dẫn đến không biết ó bao nhiêu điều kiện để task không bị **Block**.

## API Event Group trong freeRTOS

### Khởi tạo Event Group

```C
    EventGroupHandle_t xEventGroupCreate( void );
```

**Feature**:

-   Tạo một `Event Group` mới.
-   Để sử dụng được chức năng này thì RTOS API phải có: **configSUPPORT_DYNAMIC_ALLOCATION** phải được set là 1 trong `FreeRTOSConfig.h` (hoặc không cần quan tâm mặc định nó sẽ là 1).
-   Một `Event Group` được yêu cầu với lượng RAM rất nhỏ để sử dụng. Được phân bổ một cách tự động nếu sử dụng hàm `xEventGroupCreate()` và được có thể tùy biến được nếu sử dụng hàm `xEventGroupCreateStatic()`.
-   Số bit (hoặc cờ) trong một `Event Group` có thể cấu hình (xem lại phần trên).
-   Nếu một `Event Group` được tạo thành công thì một `handle` cho `Event Group` được trả về. Nếu không đủ vùng nhớ để tạo một `Event Group` thì giá trị trả về là `NULL`.

**Example**:

```C
    /* Declare a variable to hold the created event group. */
    EventGroupHandle_t xCreatedEventGroup;

    /* Attempt to create the event group. */
    xCreatedEventGroup = xEventGroupCreate();

    /* Was the event group created successfully? */
    if( xCreatedEventGroup == NULL )
    {
        /* The event group was not created because there was insufficient
        FreeRTOS heap available. */
    }
    else
    {
        /* The event group was created. */
    }
```

### Block Task để chờ 1 hoặc nhiều bit được set

```C
 EventBits_t xEventGroupWaitBits(
                       const EventGroupHandle_t xEventGroup,
                       const EventBits_t uxBitsToWaitFor,
                       const BaseType_t xClearOnExit,
                       const BaseType_t xWaitForAllBits,
                       TickType_t xTicksToWait );
```

**Feature**:

-   Đọc các bit của `Event Group` trong RTOS, sẽ được task đang sử dụng hàm này vào trạng thái **Block** (có thời gian chờ - timeout) để đợi một bit hoặc nhiều bit được thiết lập.
-   Không thể gọi này trong ngắt.

**Parameters**:

-   `xEventGroup`: **Event Group** có các bit đang được kiểm tra. **Event Group** phải được tạo bằng cách sử dụng hàm `xEventGroupCreate()`.
-   `uxBitsToWaitFor`: Các giá trị theo bit cho biết bit hoặc các biết bit cần kiểm tra bên trong **Event Group**. Ví dụ, để đợi **bit 0 and / or bit 2** thì đặt `uxBitsToWaitFor` là **0x05**.

-   `xClearOnExit`:

    -   Nếu `xClearOnExit` được đặt thành **pdTRUE** thì mọi bit được đặt trong giá trị được truyền dưới dạng tham số `uxBitsToWaitFor` sẽ bị xóa trong Event Group khi `xEventGroupWaitBits()` trả về trong khoảng thời gian chờ. Giá trị thời gian chờ được đặt trong `xTicksToWait`.
    -   Nếu `xClearOnExit` được đặt thành **pdFALSE** thì các bit ở trong Event Group sẽ không bị thay đổi khi gọi hàm và khi hàm `xEventGroupWaitBits()` trả về.

-   `xWaitForAllBits`: Được sử dụng để tạo phép thử **AND** logic (trong đó tất cả các bit phải được đặt) hoặc phép thử **OR** logic (trong đó phải đặt một hoặc nhiều bit) như sau:

    -   Nếu `xWaitForAllBits` được đặt thành **pdTRUE** thì `xEventGroupWaitBits()` sẽ trả về khi tất cả các được đặt trong giá trị được truyền dưới dạng tham số `uxBitsToWaitFor` được đặt trong **Event Group** hoặc hết thời gian được chỉ định. Có thể hiểu đơn giản đây là phép **AND**.

    -   Nếu `xWaitForAllBits` được đặt thành **pdFALSE** thì `xEventGroupWaitBits()` sẽ trả về khi bất kì bit nào được đạt trong giá trị được truyền dưới dạng tham số `uxBitsToWaitFor` được đặt trong **Event Group** hoặc hết thời gian được chỉ định. Có thể hiểu đơn giản đây là phép **OR**.

-   `xTicksToWait`: Khoảng thời gian tối đa (được tính theo `Ticks`) để chờ một hoặc tất cả (tùy thuộc vào giá trị `xWaitForAllBits`) của các bit mà `uxBitsToWaitFor` thiết lập.

**Returns**:

-   Kiếm tra giá trị trả về để biết bit nào đã được đặt. Nếu `xEventGroupWaitBits()` được trả về do hết thời gian chờ thì không phải tất cả các bit đang chờ sẽ được đặt. Nếu `xEventGroupWaitBits()` được trả về vì các bit mà nó đang chờ đã được đạt thì giá trị được trả về là giá trị của Event Group trược khi bất kỳ bit nào được tự động xóa vì tham số `xClearOnExit` được đặt thành **pdTRUE**.

**Example**:

```C
#define BIT_0	( 1 << 0 )
#define BIT_4	( 1 << 4 )

void aFunction( EventGroupHandle_t xEventGroup )
{
EventBits_t uxBits;
const TickType_t xTicksToWait = 100 / portTICK_PERIOD_MS;

  /* Wait a maximum of 100ms for either bit 0 or bit 4 to be set within
  the event group.  Clear the bits before exiting. */
  uxBits = xEventGroupWaitBits(
            xEventGroup,   /* The event group being tested. */
            BIT_0 | BIT_4, /* The bits within the event group to wait for. */
            pdTRUE,        /* BIT_0 & BIT_4 should be cleared before returning. */
            pdFALSE,       /* Don't wait for both bits, either bit will do. */
            xTicksToWait );/* Wait a maximum of 100ms for either bit to be set. */

  if( ( uxBits & ( BIT_0 | BIT_4 ) ) == ( BIT_0 | BIT_4 ) )
  {
      /* xEventGroupWaitBits() returned because both bits were set. */
  }
  else if( ( uxBits & BIT_0 ) != 0 )
  {
      /* xEventGroupWaitBits() returned because just BIT_0 was set. */
  }
  else if( ( uxBits & BIT_4 ) != 0 )
  {
      /* xEventGroupWaitBits() returned because just BIT_4 was set. */
  }
  else
  {
      /* xEventGroupWaitBits() returned because xTicksToWait ticks passed
      without either BIT_0 or BIT_4 becoming set. */
  }
}
```

### Set bits trong Event Group

```C
 EventBits_t xEventGroupSetBits( EventGroupHandle_t xEventGroup,
                                 const EventBits_t uxBitsToSet );
```

**Feature**

**Paramenter**

-   `xEventGroup`: Event Group trong đó các sẽ được thiết lập . Event Group phải được tạo bằng cách sử dụng hàm `xEventGroupCreate()`.
-   `uxBitsToSet`: Giá trị theo bit cho biết bit hoặc các bit cần thiết lập trong **Event Group**. Ví dụ, thiết lập `uxBitsToSet` là **0x08** thì chỉ có bit 3 được thiết lập. Thiết lập `uxBitsToSet` là **0x09** thì thiết lập cả bit 3 và bit 0.

**Return**

-   Giá trị của Event Group tại thời điểm hàm được gọi và trả về `xEventGroupSetBits()`.
-   Có hai lý do khiến giá trị trả về có thể vị xóa các bit do tham số `uxBitsToSet` chỉ định:

    -   Nếu việc thiết lập một bit dẫn đến một **task** thoát khỏi trạng thái **Block** thì có thể bit đó đã bị xóa tự động (xem tham số `xClearBitOnExit` của `xEventGroupWaitBits()`).
    -   Bất kỳ **task** nào được chuyển thành trạng thái Ready có mức độ ưu tiên cao hơn **task** được gọi hàm `xEventGroupSetBits()` sẽ thực thi và có thể làm thay đổi giá trị của Event Group trước khi hàm `xEventGroupSetBits()` gọi và trả về.

**Example**

```C
#define BIT_0	( 1 << 0 )
#define BIT_4	( 1 << 4 )

void aFunction( EventGroupHandle_t xEventGroup )
{
EventBits_t uxBits;

  /* Set bit 0 and bit 4 in xEventGroup. */
  uxBits = xEventGroupSetBits(
                              xEventGroup,    /* The event group being updated. */
                              BIT_0 | BIT_4 );/* The bits being set. */

  if( ( uxBits & ( BIT_0 | BIT_4 ) ) == ( BIT_0 | BIT_4 ) )
  {
      /* Both bit 0 and bit 4 remained set when the function returned. */
  }
  else if( ( uxBits & BIT_0 ) != 0 )
  {
      /* Bit 0 remained set when the function returned, but bit 4 was
      cleared.  It might be that bit 4 was cleared automatically as a
      task that was waiting for bit 4 was removed from the Blocked
      state. */
  }
  else if( ( uxBits & BIT_4 ) != 0 )
  {
      /* Bit 4 remained set when the function returned, but bit 0 was
      cleared.  It might be that bit 0 was cleared automatically as a
      task that was waiting for bit 0 was removed from the Blocked
      state. */
  }
  else
  {
      /* Neither bit 0 nor bit 4 remained set.  It might be that a task
      was waiting for both of the bits to be set, and the bits were cleared
      as the task left the Blocked state. */
  }
}

```

## Tham Khảo

> [freeRTOS - EventGroup](https://www.freertos.org/event-groups-API.html)
