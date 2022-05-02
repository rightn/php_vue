<template>
  <div class="container">

    <div class="card">
      <div class="card-header">
        Todo List
        <button class="btn btn-primary bt-write" @click="writeTodo">
          글작성
        </button>
      </div>
      <div class="card-body">
        <table class="table">
          <thead>
            <tr>
              <th>NO</th>
              <th>Title</th>
              <th>Complete</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in todos" :key="index">
              <td>{{ index + 1 }}</td>
              <td> <span @click="moveDetail(item.id)" class="detail">{{ item.title }}</span></td>
              <td>
                <input type="checkbox" class="ml-2 mr-2" :id="item.id" v-model="item.active" @change="toggleTodo(item)">
                <span class="form-check-label" :class="item.active == false ? 'active' : '' ">
                  {{ item.active ? "완료" : "진행중" }}
                </span>
              </td>
              <td>
                <div class="btn-group" role="group">
                  <button class="btn btn-primary" @click="editTodo(item.id)">수정</button>
                  <button class="btn btn-danger" @click="openModal(item.id)">삭제</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <nav aria-label="Page navigation example">
      <ul class="pagination">

        <!-- 현재페이지가 1페이지라면 보일 필요없다. -->
        <li class="page-item" v-show="page_now != 1">
          <!-- 이전페이지를 보여줌. (page_now - 1) -->
          <a class="page-link" href="#" @click="getInfo(page_now - 1)">Previous</a>
        </li>

        <li class="page-item" v-for="item in page_total" :key="item">
          <a class="page-link" href="#" @click="getInfo(item)" :class="page_now == item ? 'active' : '' ">{{item}}</a>
        </li>

        <!-- 현재페이지가 마지막 페이지라면 보일 필요없다. -->
        <li class="page-item" v-show="page_now != page_total">
          <!-- 다음페이지를 보여줌. (page_now + 1) -->
          <a class="page-link" href="#" @click="getInfo(page_now + 1)">Next</a>
        </li>
      </ul>
    </nav>

  </div>

  <!-- 경고창 -->
  <ModalWin v-if="showModal" @close="closeModal" @delete="deleteTodo" />

</template>

<script>
  import {
    ref
  } from 'vue'
  import {
    useRouter
  } from 'vue-router'
  import ModalWin from '@/components/ModalWin.vue';

  export default {
    components: {
      ModalWin
    },
    setup() {
      // 자료 보관 배열
      const todos = ref([]);
      // 서버에서 자료를 읽어오기
      const getInfo = (_page = 1) => {
        page_now.value = _page;

        fetch(`http://rightn.dothome.co.kr/data_read.php?page_now=${page_now.value}&data_count=${data_count}`)
          .then(res => res.json())
          .then(data => {
            // 배열데이터를 보관한다.    
            todos.value = data.result
            // todos 의 종류는 배열이다.
            // 배열의 for를 이용해서 접근해서
            // 만약에 각 배열의 complete 가 0, 1 이냐에 따라서
            // 그 값을 반영하는 객체를 추가한다.
            for (let item of todos.value) {
              if (item.complete === '0') {
                item.active = false;
              } else {
                item.active = true;
              }
            }
          })
          .catch()
      }



      // 모달창 닫기
      // 모달이 보여지는 상태를 저장한다.
      const showModal = ref(false);
      // 선택된 id 를 저장한다.
      const deleteId = ref(null);
      const closeModal = () => {
        // 모달창 안보이게 처리
        showModal.value = false;
        deleteId.value = null;
      }
      // 모달창 보여주기
      const openModal = (_id) => {
        // 삭제해야 하는 아이디 저장
        deleteId.value = _id;
        // 모달을 보여주고
        showModal.value = true;
      }

      // 할일 삭제 
      const deleteTodo = () => {

        fetch(`http://rightn.dothome.co.kr/data_delete.php?id=${deleteId.value}`)
          .then(res => res.json())
          .then(data => {
            // console.log(data);
            // 목록갱신
            if (data.result == 1) {

              showModal.value = false;
              deleteId.value = null;
              getInfo();

            } else {
              console.log('삭제에 실패했습니다');
            }
          })
          .catch()
      }
      // 모달 닫기
      showModal.value = false;
      // 상세보기 기능
      const router = useRouter();
      const moveDetail = (_id) => {
        // console.log(_id);
        router.push({
          name: 'Detail',
          params: {
            id: _id
          }
        });
      }


      // 수정하기
      const editTodo = (_id) => {
        router.push({
          name: 'Update',
          params: {
            id: _id
          }
        });
      }

      const writeTodo = () => {
        router.push({
          name: "Create"
        });
      }

      // 직접 구현하는 페이지네이션
      // 전체데이터 개수
      const data_total = ref(0);
      // 페이지당 보여줄 개수
      const data_count = 5;
      // 총페이지 수
      const page_total = ref(0);
      // 현재 페이지
      const page_now = ref(1);

      // 전체 데이터 수 받아오기
      const getTotal = () => {
        fetch(`http://rightn.dothome.co.kr/data_total.php`)
          .then(res => res.json())
          .then(data => {
            // 전체 데이터 수 갱신
            data_total.value = data.total;
            // 페이지 계산하기
            // 전체 페이지 갱신
            page_total.value = Math.ceil(data_total.value / data_count);
            page_now.value = 1;
            getInfo();
          })
          .catch();
      }
      getTotal();

      // 상태 업데이트
      const toggleTodo = (_obj) => {
        console.log(_obj);
        if (_obj.active == true) {
          _obj.complete = 1;
        } else {
          _obj.complete = 0;
        }

        fetch(
            `http://rightn.dothome.co.kr/data_update.php?id=${_obj.id}&title=${_obj.title}&body=${_obj.body}&complete=${_obj.complete}`
            )
          .then(res => res.json())
          .then(data => {
            console.log(data)
          })
          .catch();
      }




      return {
        todos,
        deleteTodo,
        moveDetail,
        editTodo,
        writeTodo,
        page_total,
        getInfo,
        page_now,

        closeModal,
        openModal,
        showModal,
        toggleTodo
      }
    }
  }
</script>

<style>
  .detail {
    text-decoration: underline;
    color: #000;
    cursor: pointer;
  }

  .detail:hover {
    color: hotpink;
  }

  .bt-write {
    float: right;
  }

  .active {
    background: hotpink;
    color: #fff;
  }
</style>