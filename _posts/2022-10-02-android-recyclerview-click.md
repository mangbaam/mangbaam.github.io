---
layout: post
title: Android RecyclerView 클릭 이벤트 처리는 어디서 해야할까
subtitle: Android RecyclerView Click Event better case
categories: Android
tags: [android,recyclerview]
---

## ⭐

---

리사이클러뷰를 사용할 때 클릭 이벤트를 어디서 어떻게 처리하는 것이 좋을 지 알아보자

## 간단한 예제 만들기

---

<img src="https://user-images.githubusercontent.com/44221447/193441850-be05e31d-36aa-4ca9-af09-5b44cacb5cce.gif" width="33%" />

메뉴를 선택하고 주문하기 버튼을 누르면 토스트 메시지로 주문 내용을 띄워주는 간단한 예제를 만들어보았다.

간단한 예제이기 때문에 다음 내용들은 크게 고려하지 않았다. 실제 프로젝트를 진행할 때에는 분명 신경써야 하는 부분들이다.

- string 리소스 추출
- 체크박스를 체크한 후 목록을 스크롤 한 후 돌아왔을 때 초기화 되는 상황 -> ViewHolder 가 재활용 되는 상황
- 패키지 구분
- 못생긴 UI
- 아래에 작성한 코드들은 `package`, `import` 등을 제외하고 작성함 - [저장소](https://github.com/mangbaam/MySampleApps/tree/master/recyclerviewclickbetter)에서 전체 코드 확인
- 등등

### 뷰바인딩 활성화 ( build.gradle(:app) )

```gradle
android {
    // ...

    buildFeatures {
        viewBinding = true
    }
}
```

### NoActionBar 로 변경 (theme.xml)

```xml
<resources xmlns:tools="http://schemas.android.com/tools">
    <style name="Theme.RecyclerViewClickBetter" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        // ...
    </style>
</resources>
```

### 아이템 뷰 (item_order.xml)

![image](https://user-images.githubusercontent.com/44221447/193442135-90988d8d-0fe3-464c-9d9c-936209c3c9a2.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginVertical="4dp"
    app:cardElevation="8dp"
    app:cardCornerRadius="16dp">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp">

        <TextView
            android:id="@+id/tv_table_number"
            style="@style/TextAppearance.AppCompat.Body1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textStyle="bold"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            tools:text="1 번 테이블" />

        <CheckBox
            android:id="@+id/cb_gimbob"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="김밥"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tv_table_number" />

        <CheckBox
            android:id="@+id/cb_ramen"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="라면"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/cb_gimbob" />

        <androidx.constraintlayout.widget.Barrier
            android:id="@+id/barrier"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:orientation="vertical"
            app:barrierDirection="end"
            app:barrierMargin="30dp"
            app:constraint_referenced_ids="cb_gimbob,cb_ramen" />

        <CheckBox
            android:id="@+id/cb_dduckbokki"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="떡볶이"
            app:layout_constraintStart_toEndOf="@id/barrier"
            app:layout_constraintTop_toTopOf="@id/cb_gimbob" />

        <CheckBox
            android:id="@+id/cb_cutlet"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="돈까스"
            app:layout_constraintStart_toEndOf="@id/barrier"
            app:layout_constraintTop_toTopOf="@id/cb_ramen" />

        <Button
            android:id="@+id/btn_order"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            android:text="주문하기" />
    </androidx.constraintlayout.widget.ConstraintLayout>
</androidx.cardview.widget.CardView>
```

- CardView 사용
- ConstraintLayout - Barrier 사용

### 메인 뷰 작성 (activity_main.xml)

![image](https://user-images.githubusercontent.com/44221447/193442470-623abcbb-4903-4895-8885-23632afb6de4.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.google.android.material.appbar.AppBarLayout
        android:id="@+id/appbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">

        <androidx.appcompat.widget.Toolbar
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:title="맹뱀 분식집"
            app:titleTextAppearance="@style/TextAppearance.AppCompat.Title"
            app:titleTextColor="@color/white" />
    </com.google.android.material.appbar.AppBarLayout>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rv_table"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/appbar"
        tools:listitem="@layout/item_order" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

- Custom 앱 바 사용
- RecyclerView 에 `tools:listitem="@layout/item_order"` 속성을 주면 위에서 작성한 `item_order.xml` 을 미리보기로 볼 수 있다

### 주문 모델 (Order.kt)

```kotlin
data class Order(
    val tableNumber: Int,
    val gimbob: Boolean = false,
    val ramen: Boolean = false,
    val dduckbokki: Boolean = false,
    val cutlet: Boolean = false
) {
    val valid get() = gimbob || ramen || dduckbokki || cutlet
}
```

- `valid` 는 4개의 메뉴가 모두 선택되지 않은 경우 false 값을 가지는 `getter` 이다

## MainActivity.kt 작성

---

RecyclerView 에서 아이템을 클릭하게 되면 어댑터를 통해서 해당 이벤트를 처리하게 된다. 

이전에는 `interface` 를 사용해서 콜백 형태로 넘겨주는 방식을 많이 사용했지만 코틀린으로 넘어오면서 고차함수를 통해 넘겨주는 방식으로 변화했다.

```kotlin
val orderAdapter = OrderAdapter { order ->
    if (!order.valid) {
        Toast.makeText(this, "${order.tableNumber} 번 테이블 : 주문된 음식이 없습니다", Toast.LENGTH_SHORT).show()
    } else {
        val orderInfo = getOrderInfo(order)

        Toast.makeText(this, orderInfo, Toast.LENGTH_SHORT).show()
    }
}
```

(`OrderAdapter` 는 밑에 작성했습니다)

고차함수를 통해 넘어온 `order` 를 통해 어떤 메뉴가 선택되어 있는지 판별하고 토스트 메시지를 통해 띄우는 로직을 가진다.

### 전체 코드

```kotlin
class MainActivity : AppCompatActivity() {
    lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(LayoutInflater.from(this))
        setContentView(binding.root)

        val tables = List(100) { Order(it + 1) }

        val orderAdapter = OrderAdapter { order ->
            if (!order.valid) {
                Toast.makeText(this, "${order.tableNumber} 번 테이블 : 주문된 음식이 없습니다", Toast.LENGTH_SHORT).show()
            } else {
                val orderInfo = getOrderInfo(order)

                Toast.makeText(this, orderInfo, Toast.LENGTH_SHORT).show()
            }
        }

        binding.rvTable.apply {
            layoutManager = LinearLayoutManager(
                this@MainActivity,
                LinearLayoutManager.VERTICAL,
                false
            )
            adapter = orderAdapter.also { it.items = tables }
        }
    }

    private fun getOrderInfo(order: Order): String {
        val orderInfo = StringBuilder()
            .append("${order.tableNumber} 번 테이블 : ")
        if (order.gimbob) orderInfo.append("김밥 ")
        if (order.ramen) orderInfo.append("라면 ")
        if (order.dduckbokki) orderInfo.append("떡볶이 ")
        if (order.cutlet) orderInfo.append("돈까스 ")
        orderInfo.append("주문되었습니다")

        return orderInfo.toString()
    }
}
```

## OrderAdapter 작성

---

### 일반적인 방식

```kotlin
class OrderAdapter(val orderMenu: (Order) -> Unit) :
    RecyclerView.Adapter<OrderAdapter.OrderViewHolder>() {

    var items: List<Order> = mutableListOf()
        set(value) {
            field = value
            notifyDataSetChanged()
        }

    inner class OrderViewHolder(
        private val view: ItemOrderBinding
    ) : RecyclerView.ViewHolder(view.root) {
        fun bind(order: Order) {

            // 뷰에 바인딩 처리 ...

            view.btnOrder.setOnClickListener {
                val newOrder = Order( /* ... */ ) // 현재 주문 상태
                orderMenu(newOrder) // 고차함수 사용한 콜백
            }
        }
    }

    // ...

    override fun onBindViewHolder(holder: OrderViewHolder, position: Int) {
        val item = items[position]
        holder.bind(item)
    }
}
```

- `OrderAdpater` 에서는 `orderMenu` 라고 하는 고차함수를 가지고 있다
- `ViewHolder` 의 `bind()` 메서드에서 `setOnClickListner` 를 달아서 클릭 이벤트를 처리한다
- `onBindViewHolder()` 에서 `ViewHolder.bind()` 메서드를 실행한다

위 방식은 일반적으로 많이 사용되는 패턴이다. `onBindingViewHolder` 에서 아이템 위치와 데이터를 얻어서 `ViewHolder`로 넘겨주기 쉽기 때문에 다루기 쉽고 직관적이다.

하지만 이런 방식은 성능에 최적화 된 방법은 아닐 수 있다.

RecyclerView 에서 `onBindViewHolder()` 는 `ViewHolder` 에 아이템이 바인딩 될 때마다 호출되고, `setOnClickListener` 역시 매번 트리거 될 것이다.

#### OrderAdapter.kt (전체 코드)

```kotlin
class OrderAdapter(val orderMenu: (Order) -> Unit) :
    RecyclerView.Adapter<OrderAdapter.OrderViewHolder>() {

    var items: List<Order> = mutableListOf()
        set(value) {
            field = value
            notifyDataSetChanged()
        }

    inner class OrderViewHolder(
        private val view: ItemOrderBinding
    ) : RecyclerView.ViewHolder(view.root) {
        fun bind(order: Order) {
            view.tvTableNumber.text = "${order.tableNumber} 번 테이블"
            view.cbGimbob.isChecked = order.gimbob
            view.cbRamen.isChecked = order.ramen
            view.cbDduckbokki.isChecked = order.dduckbokki
            view.cbCutlet.isChecked = order.cutlet

            view.btnOrder.setOnClickListener {
                val newOrder = Order(
                    order.tableNumber,
                    view.cbGimbob.isChecked,
                    view.cbRamen.isChecked,
                    view.cbDduckbokki.isChecked,
                    view.cbCutlet.isChecked
                )
                orderMenu(newOrder)
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): OrderViewHolder {
        val view = ItemOrderBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return OrderViewHolder(view)
    }

    override fun onBindViewHolder(holder: OrderViewHolder, position: Int) {
        val item = items[position]
        holder.bind(item)
    }

    override fun getItemCount() = items.size
}
```

### 개선된 방식

- 바인딩 될 때마다 `setOnClickListener` 를 동작시키지 않고 `ViewHolder` 가 만들어 질 때에 `setOnClickListener` 를 부착한다
- 아이템 위치는 어떻게 얻어올까? -> **`adapterPosition` 을 통해 가능하다**
- 아이템이 클릭되면 `onBindViewHolder` 에서는 `ViewHolder` 의 고차함수로 클릭된 위치를 받아 이벤트 처리
- `MainActivity` 의 코드는 변경이 없다 -> Adapter 의 고차함수로 이벤트를 넘겨주는 방식은 동일하다

![image](https://user-images.githubusercontent.com/44221447/193444288-e11bed23-47c1-4e3d-99ca-fbb91f5ba730.png)

```kotlin
class OrderAdapter(val orderMenu: (Order) -> Unit) :
    RecyclerView.Adapter<OrderAdapter.OrderViewHolder>() {

    var items: List<Order> = mutableListOf()
        set(value) {
            field = value
            notifyDataSetChanged()
        }

    inner class OrderViewHolder(
        private val view: ItemOrderBinding,
        onItemClicked: (Int) -> Unit // 클릭된 위치를 넘기기 위한 고차함수
    ) : RecyclerView.ViewHolder(view.root) {

        init {
            view.btnOrder.setOnClickListener {
                onItemClicked(adapterPosition) // 클릭된 위치 넘겨주기
            }
        }

        fun bind(order: Order) {
            // 뷰에 바인딩 처리 로직만 가짐
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): OrderViewHolder {
        val view = ItemOrderBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return OrderViewHolder(view) { index -> // 클릭된 아이템 위치
            val order = items[index]
            val newOrder = Order( /* ... */ ) // 현재 주문 상태
            orderMenu(newOrder) // 고차함수 사용한 콜백
        }
    }
}
```

#### 개선된 OrderAdapter.kt (전체 코드)

```kotlin
class OrderAdapter(val orderMenu: (Order) -> Unit) :
    RecyclerView.Adapter<OrderAdapter.OrderViewHolder>() {

    var items: List<Order> = mutableListOf()
        set(value) {
            field = value
            notifyDataSetChanged()
        }

    inner class OrderViewHolder(
        private val view: ItemOrderBinding,
        onItemClicked: (Int) -> Unit
    ) : RecyclerView.ViewHolder(view.root) {

        init {
            view.btnOrder.setOnClickListener {
                onItemClicked(adapterPosition)
            }
        }

        fun bind(order: Order) {
            view.tvTableNumber.text = "${order.tableNumber} 번 테이블"
            view.cbGimbob.isChecked = order.gimbob
            view.cbRamen.isChecked = order.ramen
            view.cbDduckbokki.isChecked = order.dduckbokki
            view.cbCutlet.isChecked = order.cutlet
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): OrderViewHolder {
        val view = ItemOrderBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return OrderViewHolder(view) { index ->
            val order = items[index]
            val newOrder = Order(
                order.tableNumber,
                view.cbGimbob.isChecked,
                view.cbRamen.isChecked,
                view.cbDduckbokki.isChecked,
                view.cbCutlet.isChecked
            )
            orderMenu(newOrder)
        }
    }

    override fun onBindViewHolder(holder: OrderViewHolder, position: Int) {
        val item = items[position]
        holder.bind(item)
    }

    override fun getItemCount() = items.size
}
```

## 결론

---

- `onBindViewHolder()` 에서 호출한 `bind()` 내부에서 `setOnClickListener` 가 존재한다면 뷰홀더가 재사용 될 때마다 트리거 된다
- `ViewHolder` 가 생성될 때 한 번만 `setOnClickListener` 를 달면 위의 문제가 해결된다
- `ViewHolder`에서 현재 몇 번째 아이템이 선택되고 있는지 알고싶다면 `adapterPosition` 을 사용하면 된다

---

## 전체 코드 (저장소)

[https://github.com/mangbaam/MySampleApps/tree/master/recyclerviewclickbetter](https://github.com/mangbaam/MySampleApps/tree/master/recyclerviewclickbetter)

## 참고한 글

[https://hamurcuabi.medium.com/recyclerview-item-click-in-a-better-way-c69d9c074ddf](https://hamurcuabi.medium.com/recyclerview-item-click-in-a-better-way-c69d9c074ddf)
