<template>
  <!--頁面整體架構 -->
  <div class="todolistbox">

    <!-- 第一個區塊：圖示、標題、按鈕 -->
    <div class="headerbox">

       <!-- 第一個區塊左側：圖示、標題-->
      <div class="header-left">
        <div>+</div>
        <h2>代辦事項</h2>
      </div>

      <!-- 第一個區塊右側：按鈕 -->
      <div class="header-right"> 
        <button class="select" @click="select_all">全選</button> <!--@click為點擊事件-->
        <button class="add" @click="to_add">添加</button>      
      </div>
    </div>

    <!-- 第二個區塊，囊括所有代辦事項內容的範圍-->
    <div class="content"> 

      <!--代辦事項裡的內容：選中、輸入框、時間、刪除按鈕-->
      <div class="content-item" v-for="(item, index) in todolist" :key="item.id" > 
      <!--單一代辦事項的格子以及格子的元素+VUE的FOR迴圈-->
      
        <!--代辦事項裡的內容左側：選中，checkbox-->
        <div class="content-left" @click="clicked(index, item.id)">
          <span :style="item.isCheck ? 'opacity:1' : 'opacity:0' ">
          </span>
        </div>

        <!--input，輸入框-->
        <input type="text" class="content-input" v-model="item.text" :disabled="item.isCheck" :class="item.isCheck ? 'cross_out' : ''" @blur="blur_and_save" :ref="el => inputBoxRefs[index] = el">

        <!--代辦事項裡的內容右側：時間、刪除按鈕 -->
        <div class="content-right">
          <p>{{ item.time }}</p> <!--{{}}為Mustache語法，是Vue模板語法的資料綁定方式之一，用來把javascript的資料顯示在HTML上-->
          <button @click="to_delete(index, item.id)">刪除</button>
        </div>

      </div>
    </div>
  </div>
</template>

<script setup>

import dayjs from 'dayjs' 
import { reactive,ref,onMounted,nextTick } from 'vue' 
//導入ref函式：建立響應式的基本型別（字串、數字、布林、陣列等）或 DOM 元素參照。當你只需要追蹤單一值（例如數字、字串、布林值）或綁定 DOM 元素。需要透過 .value來存取或修改值。

//reactive: 建立一個「響應式物件」。用來建立整個物件或陣列的響應式結構，當你有多個屬性需要追蹤變化時，使用reactive可以讓你直接操作物件的屬性而不需要 .value，Vue會自動監聽並更新畫面。
//onMounted: Vue的「生命週期鉤子」。在元件掛載完成後執行"初始化資料"、"呼叫 API"、"設定事件監聽器"等。只會執行一次，當元件第一次渲染完成時。(懶人包：用途為操作DOM元素、初始化副作用、發送API請求、執行一次性任務)
//nextTick: 在 DOM 更新完成後執行某段程式碼。當你修改資料後，想要操作更新後的 DOM（例如設定焦點、取得高度等）。Vue的更新是非同步的，確保你操作的是最新的DOM。



//ref函式建立的響應式的空陣列，設todolist為ref物件
const todolist = ref([
  // {
  // id: "1",
  // isCheck: false,
  // text:"測試1",
  // time:"輸入時間"}   ->樣板參考
])

const inputBoxRefs = ref([]) // 綁定多個input的ref(順序建議為先在javascript裡設變數->template模板裡做綁定->再回javascript裡編寫功能)
const inputLength = ref(0)   // 用來追蹤目前input的數量。ref(0)表示建立初始值為0的響應式變數

//隨機生成ID
function randomID(){
  return Math.random().toString().slice(2,8) + Date.now().toString()
  }

//讀取本地儲存的資料
onMounted(() => {
  try {
    const saved_data = localStorage.getItem("listtodo"); //設變數savedData為從瀏覽器的localStorage中讀取key為listtodo的資料。
    if (saved_data === null) {
      //如果本地沒有資料(savedData為空值)，給予預設值
      todolist.value = [{
        id: randomID(),
        text: "請點擊上方的添加按鈕添加事件", 
        isCheck: false,
        time: dayjs().format('YYYY-MM-DD HH:mm:ss')
      }];
    } else {
      // 如果本地有資料，解析saved_data的JSON字串並賦值給todolist(把JSON格式的字串轉換成 javascript 的物件或陣列)
      todolist.value = JSON.parse(saved_data);
    }
  } catch (err) {
    console.error("無法讀取本地儲存的資料", err); //console.error：錯誤訊息、異常處理。標記錯誤或異常狀況，讓你或其他開發者能快速辨識問題。 //console.log：一般訊息、除錯資訊。追蹤程式流程、變數值、函式執行等。 
    todolist.value = []; //讀取失敗時，可以給予一個空的陣列，避免白屏
  }
});

//本地儲存(自定義storage函式)
const storage = () => {
  localStorage.setItem("listtodo", JSON.stringify(todolist.value)); 
}; //將欲儲存的資料(todolist.value)轉成JSON字串()，才能儲存到 localStorage


//啟用本地儲存功能，以及判斷是否儲存至本機
function blur_and_save(){
  console.log("儲存測試")
  storage()
}//參見模板裡的input，當輸入框的游標消失時啟用storage()本地儲存功能

//添加功能
function to_add(){
  console.log("點擊了添加") //在瀏覽器的 console 顯示訊息，確認事件有觸發

  //每次添加的內容
  const newitem = {
    id: randomID(), //使用自行定義的randomID()隨機生成編號功能
    isCheck: false, //預設為false表示未選中
    text: "", //預設為空字串
    time: dayjs(new Date).format("YYYY-MM-DD HH:mm:ss") //使用dayjs時間處理函式庫，用new建立Date物件，並且將時間日期做格式化
  }

  todolist.value.unshift(newitem) //unshift：把新的項目加到陣列的最前面
  inputLength.value = todolist.value.length //儲存目前待辦事項的數量

  //新增項目後，讓第一個輸入框自動取得焦點
  nextTick(() => {
    inputBoxRefs.value[0]?.focus() //如果nputBoxRefs.value[0]存在，就呼叫它的focus()方法。
  })
}

//刪除功能
function to_delete(index, id){
  console.log("刪除測試")
  console.log(index)
  console.log(id)

  todolist.value = todolist.value.filter(item => item.id !== id) //對每一個項目item，只保留item.id不等於id的項目，把id相符的那一項「排除掉」，達到刪除的效果
  storage() //把更新後的todolist儲存於localStorage，目的為"讓使用者重新整理頁面後，資料仍然存在"、"保持畫面上的資料與本地儲存一致"以及"避免刪除後資料「回來」的情況"
}

//點擊選中功能
function clicked(index, id){
  console.log("選中測試")
  console.log(index)
  console.log(id)

  todolist.value[index].isCheck = !todolist.value[index].isCheck //找出代辦清單中第index筆資料，然後把它的isCheck屬性反轉（true ↔ false）
  storage()
}

//全選功能(如果所有項目都處於選中的情況下點擊全選則為取消全選)
function select_all(){
  console.log("全選測試")

  const allChecked = todolist.value.every(item => item.isCheck) //every:用來檢查「是不是每一項都符合某個條件」
  todolist.value.forEach(item => {
    item.isCheck = !allChecked
  })
  //逐一將各項目的選中狀態改為allChecked的反轉值
  //如果全部未選或者部分未選(allChecked為false)，則選中狀態(item.isCheck為false)變為allChecked的相反，true
  //如果全選(allChecked為true)，則選中狀態(item.isCheck為true)變為allChecked的相反，false
  storage()
}

</script>

<style scoped>

.todolistbox{
  width: 720px;
  height: 540px;
  background-color: slategrey;
  border-radius: 10px;
  position: absolute; /* 使用絕對定位，讓元素可以自由擺放在畫面上 */
  top: 50%; /* 從畫面頂部開始往下 50% 的位置 */
  left: 50%; /* 從畫面左側開始往右 50% 的位置 */
  transform: translate(-50%,-50%); /* 往左上移動自身寬高的一半，達到真正置中效果 */
  padding: 20px; /* 內邊距 20px，讓內容不會貼邊 */
  box-sizing: border-box; /* 包含 padding 在內計算寬高，避免內容溢出 */
  color: azure;

  .headerbox{
    display: flex;
    justify-content: space-between; /*左右分散排列：讓.header-left(+)和.header-right(全選、添加)分佈在容器兩側 */
    align-items: center; /* 垂直置中：讓子元素在容器的垂直方向上置中對齊*/
    border-bottom: black 1px solid; /*下邊框*/
    }

    .header-left{
      display: flex;  /*使用 Flexbox 排版，讓裡面的元素可以橫向排列(+ 和 h2 橫向排列)*/
      align-items: center; /*垂直置中裡面的元素（例如那個圓形圖示）*/

    /*+號圖形位於header-left裡，故使用巢狀結構*/
      div{
        width: 50px; /*設定寬度為 50px*/
        height: 50px; /*設定高度為 50px（與寬度相同，方便做成圓形）*/
        border-radius: 50%; /*圓角設為 50%，讓這個 div 呈現為圓形*/
        font-size: 30px;
        background-color: #9999E6;
        text-align: center;
        margin-right: 15px;

        display: flex;
        justify-content: center;
        align-items: center;
        }
      }

    .header-right{
      display: flex;
      justify-content: flex-end; /* 讓按鈕靠右排列 */
      flex-grow: 1; /* 撐滿剩下的空間 */
      gap: 15px; /*gap：控制整個容器中元素的間距*/
    
      .select{
        background-color:  #C43F38;
        }

      .add{
        background-color:  #4BB15C;
        }
      }

  .content{
    margin-top: 20px;
    height: 400px;
    overflow-y: scroll;

    .content-item{
      display: flex;
      justify-content: space-between; /* 元素平均分散，首尾元素貼齊邊界*/
      background-color: #5E6372;
      align-items: center;
      border-radius: 10px;
      padding: 10px 10px; /*padding:內容與邊框之間的空間*/
      margin-top: 10px; /*margin:元素與其他元素之間的距離*/
      box-sizing: border-box; /*容器大小固定 */

      .content-left{
        width: 30px;
        height: 30px;
        border: 1px solid white;
        border-radius: 50%;
        position: relative;

        /* checkbox的設定*/
        span{
          display: inline-block;
          width: 20px;
          height: 20px;
          background-color: #01ff2b;
          border-radius: 50%;
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%,-50%);
        }
      }

      .content-input{
        flex: 1;
        /*
        felx:1的含意->
        flex-grow: 1;
        flex-shrink: 1;
        flex-basis: 0;
        功能為撐滿剩下的空間
        */
        margin: 0 10px;
        outline: none; /*去除選中效果 */
        background-color: transparent;
        border: none; /*輸入框變透明後仍然會留有框，這一行可將框去除 */
        border-bottom: 1px white solid;
        padding: 5px 10px;
        color: white;
        }
      
      .cross_out{
        color: rgba(255, 255, 255, 0.7); /*文字顏色與透明度*/
        text-decoration: line-through rgba(255, 255, 255, 0.7); /*橫線顏色與透明度*/
      }

      .content-right{
        display: flex;
        align-items: center;
        gap: 10px; /*設定子元素之間的水平間距 */
        button{
          background-color: #C43F38;
        }

      }
    }
    }
  }

button{
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  color: white;
  }

</style>