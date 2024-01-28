---
title: 【AndroidView】RecyclerView から Groupi を使ってみる
tags:
  - Android
  - Groupie
  - Kotlin
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
今まで RecyclerView で実装していたことを Groupi で実装することがあったので、そのメモです。

# 導入
Groupy のリポジトリにある [Try it out](https://github.com/lisawray/groupie?tab=readme-ov-file#try-it-out) の通りに導入します。

Gradle に implementation を追加します。
```gradle: build.gradle
dependencies {
    implementation "com.github.lisawray.groupie:groupie:$groupie_version"
}
```
次に、settings.gradleに `maven { url 'https://jitpack.io' }` を追加します。最新のプロジェクトではこれがないと動かないようです。(実際、自分も動きませんでした。)
```diff_gradle: settings.gradle
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
+       maven { url 'https://jitpack.io' } 
    }
}
```

他、ViewBinding、DataBinding を使っている場合は以下も implementation に追加します。

```diff_gradle: build.gradle
dependencies {
    implementation "com.github.lisawray.groupie:groupie:$groupie_version"
+   implementation "com.github.lisawray.groupie:groupie-viewbinding:$groupie_version" 
}
```

# 置き換え
変更前のコードは以下のようになります。

```kotlin: MainActivity.kt
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        setupRecyclerView()
    }

    private fun setupRecyclerView() {
        val items = listOf("🍎りんご", "🍌バナナ", "🍒さくらんぼ", "🍊オレンジ", "🍓いちご", "🍉すいか", "🍐洋梨")
        binding.mainRecyclerView.adapter = ItemAdapter(items)
        binding.mainRecyclerView.layoutManager = LinearLayoutManager(this)
    }
}
```
```kotlin: ItemAdapter.kt
class ItemAdapter(private val items: List<String>): RecyclerView.Adapter<ItemAdapter.ItemViewHolder>() {
    class ItemViewHolder(val binding: RecyclerItemBinding) : RecyclerView.ViewHolder(binding.root)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ItemViewHolder {
        val binding = RecyclerItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return ItemViewHolder(binding)
    }

    override fun getItemCount(): Int {
        return items.size
    }

    override fun onBindViewHolder(holder: ItemViewHolder, position: Int) {
        holder.binding.textViewItem.text = items[position]
    }
}
```

```xml:activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/main_recycler_view"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        />
</androidx.constraintlayout.widget.ConstraintLayout>
```

```xml:recycler_item.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:padding="50dp"
    >

    <TextView
        android:id="@+id/text_view_item"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="@string/sample_title"
        android:textSize="30sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        />

</androidx.constraintlayout.widget.ConstraintLayout>
```

ここで、以下のような変更を加えると、Groupieで実装したことになります。

```diff_kotlin: MainActivity.kt
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
+   private val groupAdapter = GroupAdapter<GroupieViewHolder>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        setupRecyclerView()
    }

    private fun setupRecyclerView() {
        val items = listOf("🍎りんご", "🍌バナナ", "🍒さくらんぼ", "🍊オレンジ", "🍓いちご", "🍉すいか", "🍐洋梨")
-       binding.mainRecyclerView.adapter = ItemAdapter(items)
+       binding.mainRecyclerView.adapter = groupAdapter
        binding.mainRecyclerView.layoutManager = LinearLayoutManager(this)
+       groupAdapter.addAll(items.map {
+             ItemAdapter(it)
+         })
    }
}
```

`ItemAdapter` はごっそり変更します。
便宜上、`ItemAdapter` となっていますが、`Adapter` ではなく `○○Item`などの方が良いです。

```kotlin: ItemAdapter
class ItemAdapter(private val item: String): Item<GroupieViewHolder>() {
    override fun bind(viewHolder: GroupieViewHolder, position: Int) {
        val binding = RecyclerItemBinding.bind(viewHolder.itemView)
        binding.textViewItem.text = item
    }

    override fun getLayout() = R.layout.recycler_item
}
```

# 最後に
サンプルアプリも作成してみました。もし、間違っている箇所や、こうした方が良いという箇所があればPR待っています！

https://github.com/KaitoMuraoka/RecyclerViewToGroupieSampleApp

# 参考文献
 https://github.com/lisawray/groupie?tab=readme-ov-file
 
 https://medium.com/@soosyamoora/simplifying-recyclerview-using-groupie-19ef361e52b5
