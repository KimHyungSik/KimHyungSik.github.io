---
layout: post
title:  안드로이드 Paging CleanArchitecture 적용하기
categories: [Android, CleanArchitecture, Paging]
tags: [Android, CleanArchitecture, Paging, Andrid JetPack]
comments: true 
---

Paging3
=====
[Android Developers](https://developer.android.com/topic/libraries/architecture/paging/v3-overview)
>로컬 저장소에서나 네트워크를 통해 대규모 데이터 세트의 데이터 페이지를 로드하고 표시할 수 있습니다

Paging 라이브러리는 페이징을 통해 데이터를 가져올때 더욱 효율적이고 간편하게 동작할 수 있도록 구현된 Android JetPack라이브러리 입니다. 
페이징을 통해서 얻을수 있는 장점은 
> - 페이징된 데이터의 메모리 내 캐싱. 이렇게 하면 앱이 페이징 데이터로 작업하는 동안 시스템 리소스를 효율적으로 사용할 수 있습니다.
> - 요청 중복 제거 기능이 기본으로 제공되어 앱에서 네트워크 대역폭과 시스템 리소스를 효율적으로 사용할 수 있습니다.
> - 사용자가 로드된 데이터의 끝까지 스크롤할 때 구성 가능한 RecyclerView 어댑터가 자동으로 데이터를 요청합니다.
> - Kotlin 코루틴 및 Flow뿐만 아니라 LiveData 및 RxJava를 최고 수준으로 지원합니다.
새로고침 및 재시도 기능을 포함하여 오류 처리를 기본으로 지원합니다.


Dependencies
-----
```
def paging_version = "3.1.0"

implementation "androidx.paging:paging-runtime:$paging_version"
```

기본적인 구현에는 paging dependencise만 있으면 됩니다.
하지만 이번에는 Clean Architectue로 구현할 예정이기 떄문에 

```
// alternatively - without Android dependencies for tests
testImplementation "androidx.paging:paging-common:$paging_version"
```
Android에 종속성이 없는 dependencies를 domain layer에 추가 합니다.

Data layer 구현
----
먼저 Data를 호출하는 동작은 Data layer에 PagingSource 구현해야 합니다. 
```kt
class SearchBookSource(
    private val searchBookService: SearchBookService,
    private val query: String
) :PagingSource<Int, BookListItem>() {
    override fun getRefreshKey(state: PagingState<Int, BookListItem>): Int? {
        return state.anchorPosition?.let { anchorPosition ->
            state.closestPageToPosition(anchorPosition)?.prevKey?.plus(1)
                ?: state.closestPageToPosition(anchorPosition)?.nextKey?.minus(1)
        }
    }

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, BookListItem> {
        return try {
            // 최초 실행시 page는 null입니다.
            val page = params.key ?: 1
            // 실질적으로 데이터를 호출합니다.
            val response = searchBookService.searchBookRequest(query, page)
            LoadResult.Page(
                // 호추한 데이터를 반환하여 UI에 데이터를 보냅니다.
                data = response.books,
                // 이전 페이지
                prevKey = if (page == 1) null else page -1,
                // 다음 페이지를 구현할때 api의 조건에 따라 다음 페이지가 없다면 null을 넣으면 됩니다.
                nextKey = if (response.total.toInt() == 0) null else response.page.toInt() + 1
            )
        }catch (e: Exception){
            LoadResult.Error(e)
        }
    }
}
```

다음으로 PagingSource를 사용할 수 있도록 Repository에 구현합니다. (Hilt를 사용하여 service를 주입 받았습니다.)
```kt
class SearchBooksRepositoryImp @Inject constructor (private val searchBookService: SearchBookService) : SearchBooksRepository {
    override fun getSearchBooksByPaging(query: String): Flow<PagingData<BookListItem>> {
        return Pager(
            // 화면에 보여줄 데이터의 개수 입니다.
            config = PagingConfig(pageSize = 10),
            pagingSourceFactory = {SearchBookSource(searchBookService, query)}
            // flow를 사용하여 구현했습니다
        ).flow // liveData또한 지원하기 떄문에 .liveData를 이용하여 liveData형태로 반환 가능합니다.
    }
}
```

Domain layer 구현
----
Domain layer에서는 Android에 의존하는 라이브러리를 사용하면 안됩니다.
그래서 Android에 의존성이 없는 Dependencies를 호출해야 합니다.
```
// alternatively - without Android dependencies for tests
testImplementation "androidx.paging:paging-common:$paging_version"
```
Repository구현체에서 PagingSource구현체를 받아 useCase에 전달 합니다.
```kt
class SearchBooksUseCase(private val repo: SearchBooksRepository) {
    fun paging(query: String) : Flow<PagingData<BookListItem>> {
        return repo.getSearchBooksByPaging(query)
    }
```

Presentation layer 구현
-----
먼저 RecyclerView를 구현 합니다. RecyclerView를 구현 할때는 RecyclerView.Adapter가 아닌 PagingDataAdapter를 이용하여 구현해야 합니다.
```kt
class BookListAdapter(private val itemClickListener: (isbn13: String, bitmap: Bitmap) -> Unit) :
    PagingDataAdapter<BookListItem, BookListAdapter.BookListHolder>(diffUtil) {
    inner class BookListHolder(
        private val binding: ItemBookListBinding
    ) : RecyclerView.ViewHolder(binding.root) {
        fun binding(bookListItem: BookListItem) {
            ...
        }
    }

    companion object {
        val diffUtil = object : DiffUtil.ItemCallback<BookListItem>() {
           ...
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): BookListHolder =
        BookListHolder(
            ...
        )

    override fun onBindViewHolder(holder: BookListHolder, position: Int) {
        getItem(position)?.let { holder.binding(it) }
    }
}
```

RecyclerView구현이 완료 되었다며 ViewModel에서 데이터를 호출하여 RecyclerView에 데이터를 주입하기만하면 됩니다.
```kt
// ViewModel
private fun searchBooks(query: String) {
    viewModelScope.launch {
        searchBooksUseCase.paging(query).cachedIn(viewModelScope).collectLatest {
            _searchBooks.value = it
        }
    }
}
//Fragment
        vm.bookListLiveData.observe(viewLifecycleOwner) { pagingData ->
            viewLifecycleOwner.lifecycleScope.launch {
                bookListAdapter.submitData(pagingData)
            }
        }
```
