<template>
  <div class="container">
    <div class="row">
      <div class="col-md-6">
        <button type="button" class="btn btn-primary mb-3" style="float: left;" @click="fnWrite">등록</button>
      </div>
      <div class="col-md-6">
        <div class="input-group mb-3">
          <select class="form-select" v-model="searchKey">
            <option value="">- 선택 -</option>
            <option value="author">작성자</option>
            <option value="title">제목</option>
            <option value="contents">내용</option>
          </select>
          <input type="text" class="form-control" v-model="searchValue" @keyup.enter="fnPage()">
          <button class="btn btn-outline-secondary" type="button" @click="fnPage()">검색</button>
        </div>
      </div>
    </div>
    <table class="table">
      <thead>
        <tr>
          <th scope="col">No</th>
          <th scope="col">제목</th>
          <th scope="col">작성자</th>
          <th scope="col">등록일시</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, id) in list" :key="id">
          <td>{{ row.id }}</td>
          <td><a @click="fnView(row.id)" style="cursor: pointer;">{{ row.title }}</a></td>
          <td>{{ row.nickName }}</td>
          <td>{{ row.createDate }}</td>
        </tr>
      </tbody>
    </table>

    <div class="pagination d-flex justify-content-center" v-if="paging.totalListCnt > 0">
      <ul class="pagination">
        <li class="page-item" :class="{ disabled: paging.page === 1 }">
          <a class="page-link" href="#" @click.prevent="fnPage(1)">처음</a>
        </li>
        <li class="page-item" :class="{ disabled: paging.page === 1 }">
          <a class="page-link" href="#" @click.prevent="fnPage(paging.page - 1)">이전</a>
        </li>
        <li class="page-item" v-for="n in paginavigation()" :key="n" :class="{ active: paging.page === n }">
          <a class="page-link" href="#" @click.prevent="fnPage(n)">{{ n }}</a>
        </li>
        <li class="page-item" :class="{ disabled: paging.page === paging.totalPage }">
          <a class="page-link" href="#" @click.prevent="fnPage(paging.page + 1)">다음</a>
        </li>
        <li class="page-item" :class="{ disabled: paging.page === paging.totalPage }">
          <a class="page-link" href="#" @click.prevent="fnPage(paging.totalPage)">마지막</a>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() { //변수생성
    return {
      requestBody: {}, //리스트 페이지 데이터전송
      list: {}, //리스트 데이터
      no: '', //게시판 숫자처리
      memberInfo: null,

      paging: {
        block: 0,
        endPage: 0,
        nextBlock: 0,
        page: 0,
        pageSize: 0,
        prevBlock: 0,
        startIndex: 0,
        startPage: 0,
        totalBlockCnt: 0,
        totalListCnt: 0,
        totalPageCnt: 0,
      }, //페이징 데이터
      page: this.$route.query.page ? this.$route.query.page : 1,
      size: this.$route.query.size ? this.$route.query.size : 10,
      searchKey: this.$route.query.sk ? this.$route.query.sk : '',
      searchValue: this.$route.query.sv ? this.$route.query.sv : '',
      paginavigation: function () { //페이징 처리 for문 커스텀
        let pageNumber = [] //;
        let startPage = this.paging.startPage;
        let endPage = this.paging.endPage;
        for (let i = startPage; i <= endPage; i++) pageNumber.push(i);
        return pageNumber;
      }
    }
  },
  mounted() {
    this.fnGetList()
  },
  methods: {
    fnGetList() {
      this.requestBody = { // 데이터 전송
        sk: this.searchKey,
        sv: this.searchValue,
        page: this.page,
        size: this.size
      }
      this.$axios.get(this.$serverUrl + "/api/board/list", {
        params: this.requestBody,
        headers: {}
      }).then((res) => {
        this.list = res.data  //서버에서 데이터를 목록으로 보내므로 바로 할당하여 사용할 수 있다.

        if (res.data.resultCode === "OK") {
          this.list = res.data.data
          this.paging = res.data.pagination
          this.no = this.paging.totalListCnt - ((this.paging.page - 1) * this.paging.pageSize)
        }

      }).catch((err) => {
        if (err.message.indexOf('Network Error') > -1) {
          alert('네트워크가 원활하지 않습니다.\n잠시 후 다시 시도해주세요.')
        }
      })
    },
    fnView(id) {
      this.requestBody.id = id
      this.$router.push({
        path: './boardDetail',
        query: this.requestBody
      })
    },
    fnWrite() {
      this.$router.push({
        path: './boardWrite'
      })
    },
    fnPage(n) {
      if (this.page !== n) {
        this.page = n
      }

      this.fnGetList()
    }
  }
}
</script>