<template>
  <div>
    <a-page-header
      :ghost="false"
      @back="() => $router.go(-1)"
      title="Gantt"
      :breadcrumb="breadcrumb"
      class="gantt-header"
    />
    <div class="gantt-page" :style="{ margin: '24px' }">
          <a-card
              :bordered="false"
              :title="`Sơ đồ Gantt - ${resultInitDepartmentGanttPage?.departments.data.name} - ${thisFullYear}`"
          >
            <div :style="{ display: 'flex', justifyContent: 'center'}">
              <a-button type="primary" @click="showModal">
               <TableOutlined />
                Sơ đồ Gantt
              </a-button>
              <a-modal
                  v-model:visible="visible"
                  :title="`Sơ đồ Gantt - ${resultInitDepartmentGanttPage?.departments.data.name} - ${thisFullYear}`"
                  width="100%"
                  wrap-class-name="full-modal"
                  @ok="handleOk"
                  :footer=null
              >
                <div :style="{ float: 'left' }">
                  <a-button type="primary" @click="minimizeTaskList">Thu nhỏ danh sách Task</a-button>
                </div>
                <div :style="{ display: 'flex', justifyContent: 'flex-end', marginBottom: '10px' }">

                      <div :style="{ marginRight: '15px' }">
                        <a-select
                            v-model:value="dataEmployee"
                            show-search
                            placeholder="Vui lòng chọn nhân viên"
                            style="width: 200px"
                            :options="optionEmployee"
                            :filter-option="filterOptionEmployee"
                            @change="handleChangeEmployee"
                        />
                      </div>
                      <div :style="{ marginRight: '15px' }">
                        <a-select
                            v-model:value="dataTimeRange"
                            show-search
                            placeholder="Vui lòng chọn thời gian"
                            style="width: 200px"
                            @select="selectTimeRange"
                        >
                          <a-select-option value="Tuần này">Tuần này</a-select-option>
                          <a-select-option value="Tháng này">Tháng này</a-select-option>
                          <a-select-option value="Năm này">Năm này</a-select-option>
                        </a-select>
                      </div>
                      <a-range-picker
                          :value="hackValue || value"
                          @change="onChange"
                          @openChange="onOpenChange"
                          @calendarChange="onCalendarChange"
                      />
                      <div class="processDropdown">
                        <a-dropdown key="more">
                          <a-button type="link"
                                    :style="{
                          fontSize: '30px',
                          paddingRight: 0,
                          display: 'flex',
                          alignItems: 'center',
                        }"
                          >
                            <MenuOutlined />
                          </a-button>
                          <template #overlay>
                            <a-menu>
                              <a-menu-item>
                                <NuxtLink target="_blank" rel="noopener noreferrer" href="#">
                                  <span>
                                    <UserSwitchOutlined :style="{ paddingRight: '8px' }" />
                                    Bàn giao công việc
                                  </span>
                                </NuxtLink>
                              </a-menu-item>
                              <a-menu-item>
                                <NuxtLink target="_blank" rel="noopener noreferrer" href="#">
                                  <span>
                                    <SendOutlined :style="{ paddingRight: '8px' }" />
                                    Gửi phê duyệt
                                  </span>
                                </NuxtLink>
                              </a-menu-item>
                              <a-menu-item>
                                <NuxtLink target="_blank" rel="noopener noreferrer" href="#">
                                  <span>
                                    <MergeCellsOutlined :style="{ paddingRight: '8px' }" />
                                    Kiểm tra dữ liệu trùng và gộp
                                  </span>
                                </NuxtLink>
                              </a-menu-item>
                              <a-divider :style="{ left: 0, width: '100%', marginTop: '8px', marginBottom: '8px' }"></a-divider>
                              <a-menu-item>
                                <NuxtLink target="_blank" rel="noopener noreferrer" href="#">
                                  <span>
                                    <CopyOutlined :style="{ paddingRight: '8px' }" />
                                    Nhân bản
                                  </span>
                                </NuxtLink>
                              </a-menu-item>
                              <a-menu-item>
                                <NuxtLink target="_blank" rel="noopener noreferrer" href="#">
                                  <span>
                                    <ShareAltOutlined :style="{ paddingRight: '8px' }" />
                                    Chia sẻ
                                  </span>
                                </NuxtLink>
                              </a-menu-item>
                              <a-menu-item>
                                <span @click="exportImg">
                                  <FileImageOutlined :style="{ paddingRight: '8px' }" />
                                  Xuất ảnh
                                </span>
                              </a-menu-item>
                              <a-menu-item>
                                <span @click="exportGanttExcel">
                                  <FileExcelOutlined :style="{ paddingRight: '8px' }" />
                                  Xuất Excel
                                </span>
                              </a-menu-item>
                              <a-menu-item>
                                <NuxtLink target="_blank" rel="noopener noreferrer" href="#">
                                  <span>
                                    <PrinterOutlined :style="{ paddingRight: '8px' }" />
                                    In
                                  </span>
                                </NuxtLink>
                              </a-menu-item>
                              <a-menu-item>
                                <NuxtLink target="_blank" rel="noopener noreferrer" href="#">
                                  <span>
                                    <DeleteOutlined :style="{ paddingRight: '8px' }" />
                                    Xóa
                                  </span>
                                </NuxtLink>
                              </a-menu-item>
                            </a-menu>
                          </template>
                        </a-dropdown>
                      </div>
                    </div>
                <Gantt
                    ref="gantt"
                    :data="dataGantt"
                    itemText=""
                    dateText=""
                    :dateRangeList="dateRangeList"
                    itemWidth="40"
                    itemHeight="50"

                />
              </a-modal>
            </div>
          </a-card>
    </div>
  </div>
</template>

<script setup>
import {ref} from 'vue';
import dayjs from "dayjs";
import Gantt from 'vue3-gantt';
import 'vue3-gantt/dist/style.css';
import {
  TableOutlined,
  FileImageOutlined,
  FileExcelOutlined,
  MenuOutlined,
  UserSwitchOutlined,
  SendOutlined,
  MergeCellsOutlined,
  CopyOutlined,
  ShareAltOutlined,
  PrinterOutlined,
  DeleteOutlined,
} from '@ant-design/icons-vue';

const breadcrumb = {
  routes: [
    {
      path: '/erp/task',
      breadcrumbName: 'Tổng quan công việc',
    },
    {
      path: '/erp/task/gantt',
      breadcrumbName: 'Sơ đồ công việc Gantt',
    },
  ],
};
const router = useRouter();
const route = useRoute();
const id = route.params.id;

// MODAL GANTT
// Department:
const queryInitDepartmentGanttPage = gql`
  query departments($first: Int) {
        departments(
          first: $first
    ) {
      paginatorInfo {
        count
      }
      data {
        id
        code
        name
      }
    }
  }
`;
const { result: resultInitDepartmentGanttPage } = useQuery(
    queryInitDepartmentGanttPage,
    {
      fetchPolicy: 'no-cache',
    },
);
//  Modal:
const visible = ref(false);
const showModal = () => {
  visible.value = true;
  changeWeekDays();
};
// Gantt Chart
// Buttons
let element = document.getElementsByClassName('guide');
const minimizeTaskList = (e) => {
  e.target.className += 'minimize';
};
const gantt = ref(null);
const exportImg = () => {
  gantt.value.exportImg({ download: true, waterValue: 'Glodival Gantt Chart' })
};
const exportGanttExcel = () => {
  gantt.value.exportGanttExcel({ fileName: 'Glodival Gantt Chart' })
};
// Select Time Range
const thisDay = dayjs().format('d');
const thisDate = dayjs().format('DD');
const thisMonth = dayjs().format('MM');
const thisFullYear = dayjs().format('YYYY');
const thisLastDate = `${thisFullYear}-${thisMonth}-31`;
const value = ref();
const dates = ref();
const hackValue = ref();
const dataTimeRange = ref();
const selectTimeRange = () => {
  if(dataTimeRange.value==='Tuần này'){
    hackValue.value = [
      dayjs(`${thisFullYear}-${thisMonth}-${thisDate-thisDay+1}`),
      dayjs(`${thisFullYear}-${thisMonth}-${thisDate-thisDay+7}`)
    ]
  } else if(dataTimeRange.value==='Tháng này'){
    hackValue.value = [
      dayjs(`${thisFullYear}-${thisMonth}-01`),
      dayjs(`${thisFullYear}-${thisMonth}-${31-dayjs(thisLastDate).format('DD')%31}`)
    ]
  } else {
    hackValue.value = [dayjs(`${thisFullYear}-01-01`), dayjs(`${thisFullYear}-12-31`)]
  }
}
const onOpenChange = open => {
  if (open) {
    dates.value = [];
    hackValue.value = [];
  } else {
    hackValue.value = undefined;
  }
};
const onChange = (val) => {
  value.value = val;
  console.log('onChange', val);
};
const onCalendarChange = val => {
  dates.value = val;
  console.log('onCalendarChange', val);
};
// Select Employee
const dataEmployee = ref();
let optionEmployee = ref();
const queryInitUserAttendancePage = gql`
  query users($first: Int) {
    users(
      first: $first
    ) {
      data {
        id
        name
      }
    }
  }
`;
const { onResult: onResultInitUser } = useQuery(
    queryInitUserAttendancePage,//
    {
      fetchPolicy: 'no-cache',
    },
);
onResultInitUser(({ data }) => {
  if (data?.users) {
    optionEmployee.value = data?.users.data.map(user => ({
      label: user.name,
      value: user.name,
    }));
  }
});
const handleChangeEmployee = dataEmployee => {
  variables.userId = `${dataEmployee.value}`;
  refetch(variables);
  console.log('Đang chọn', `${dataEmployee}`);
};
const filterOptionEmployee = (input, option) => {
  return option.value.toLowerCase().indexOf(input.toLowerCase()) >= 0;
};
// Task List
const dateRangeList = ref(['2023-01-01', '2023-12-31'])
const dataGantt = ref([
  {
    type: 'normal',
    color: '',
    name: 'SPRINT 1',
    schedule: [{
      id: 112,
      name: '(1/1 - 7/1)',
      desc: 'Sprint 1',
      backgroundColor: '#B20000',
      textColor: '#fff',
      days: ["2023-01-01","2023-01-07"],
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'Page chấm công',
    schedule: [{
        id: 11,
        name: 'Miguel ',
        desc: '2/1 - 3/1',
        backgroundColor: '#CC0000',
        textColor: '#fff',
        days: ["2023-01-02","2023-01-03"],
      }],
  },
  {
    type: 'normal',
    color: '',
    name: 'Page quản lý tài sản',
    schedule: [{
        id: 21,
        name: 'Tam',
        desc: '2023/01/08 - 2023/01/14',
        backgroundColor: '#E50000',
        textColor: '#fff',
        days: ["2023-01-02","2023-01-04"]
      }],
  },
  {
    type: 'normal',
    color: '',
    name: 'Page đánh giá nhân viên',
    schedule: [{
        id: 21,
        name: 'Huyen',
        desc: '2023/01/08 - 2023/01/07',
        backgroundColor: '#FF0000',
        textColor: '#fff',
        days: ["2023-01-03","2023-01-07"]
      }],
  },
  {
    type: 'normal',
    color: '',
    name: 'Page đánh giá phòng ban',
    schedule: [
      {
        id: 21,
        name: 'Miguel ',
        desc: '2/1 - 3/1',
        backgroundColor: '#CC0000',
        textColor: '#fff',
        days: ["2023-01-04","2023-01-06"],
      }
    ],
  },
  {
    type: 'normal',
    color: '',
    name: 'SPRINT 2',
    schedule: [{
      id: 112,
      name: '(8/1 - 14/1)',
      desc: 'Sprint 2',
      backgroundColor: 'blue',
      textColor: '#fff',
      days: ["2023-01-08","2023-01-14"],
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'UI Desktop - Page đăng ký email',
    schedule: [{
        id: 21,
        name: 'Miguel',
        desc: '2023/01/08 - 2023/01/14',
        backgroundColor: 'blue',
        textColor: '#fff',
        days: ["2023-01-08","2023-01-10"]
      }],
  },
  {
    type: 'normal',
    color: '',
    name: 'API - Page đăng ký email',
    schedule: [{
      id: 21,
      name: 'Max',
      desc: '2023/01/08 - 2023/01/14',
      backgroundColor: 'blue',
      textColor: '#fff',
      days: ["2023-01-08","2023-01-10"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'Call API - Page đăng ký email',
    schedule: [{
      id: 21,
      name: 'Miguel',
      desc: '2023/01/08 - 2023/01/14',
      backgroundColor: 'blue',
      textColor: '#fff',
      days: ["2023-01-08","2023-01-10"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'SPRINT 3',
    schedule: [{
      id: 112,
      name: '(15/1 - 21/1)',
      desc: 'Sprint 3',
      backgroundColor: 'green',
      textColor: 'white',
      days: ["2023-01-15","2023-01-21"],
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'UI Desktop - Page task Gantt',
    schedule: [{
      id: 21,
      name: 'Miguel',
      desc: '2023/01/19 - 2023/01/21',
      backgroundColor: 'green',
      textColor: 'white',
      days: ["2023-01-19","2023-01-21"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'API - Page task Gantt',
    schedule: [{
      id: 21,
      name: 'Max',
      desc: '2023/01/17 - 2023/01/21',
      backgroundColor: 'green',
      textColor: 'white',
      days: ["2023-01-17","2023-01-21"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'Call API - Page task Gantt',
    schedule: [{
      id: 21,
      name: 'Miguel',
      desc: '2023/01/19 - 2023/01/20',
      backgroundColor: 'green',
      textColor: 'white',
      days: ["2023-01-19","2023-01-20"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'SPRINT 4',
    schedule: [{
      id: 112,
      name: '(22/1 - 28/1)',
      desc: 'Sprint 4',
      backgroundColor: 'gold',
      textColor: 'white',
      days: ["2023-01-22","2023-01-28"],
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'UI Desktop - Page payment',
    schedule: [{
      id: 21,
      name: 'Huyen',
      desc: '2023/01/22 - 2023/01/25',
      backgroundColor: 'gold',
      textColor: 'white',
      days: ["2023-01-22","2023-01-25"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'API - Page payment',
    schedule: [{
      id: 21,
      name: 'Hai',
      desc: '2023/01/24 - 2023/01/27',
      backgroundColor: 'gold',
      textColor: 'white',
      days: ["2023-01-24","2023-01-27"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'Call API - Page payment',
    schedule: [{
      id: 21,
      name: 'Huyen',
      desc: '2023/01/26 - 2023/01/28',
      backgroundColor: 'gold',
      textColor: 'white',
      days: ["2023-01-26","2023-01-28"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'SPRINT 5',
    schedule: [{
      id: 112,
      name: '(29/1 - 4/2)',
      desc: 'Sprint 2',
      backgroundColor: 'violet',
      textColor: 'white',
      days: ["2023-01-29","2023-02-04"],
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'UI Desktop - Page đăng ký thành viên',
    schedule: [{
      id: 21,
      name: 'Tam',
      desc: '2023/01/29 - 2023/01/31',
      backgroundColor: 'violet',
      textColor: 'white',
      days: ["2023-01-29","2023-01-31"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'API - Page đăng ký email',
    schedule: [{
      id: 21,
      name: 'Wilson',
      desc: '2023/02/01 - 2023/02/03',
      backgroundColor: 'violet',
      textColor: 'white',
      days: ["2023-02-01","2023-02-03"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'Call API - Page đăng ký thành viên',
    schedule: [{
      id: 21,
      name: 'Tam',
      desc: '2023/02/02- 2023/02/04',
      backgroundColor: 'violet',
      textColor: 'white',
      days: ["2023-02-02","2023-02-04"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'SPRINT 52',
    schedule: [{
      id: 112,
      name: '(24/12 - 30/12)',
      desc: 'Sprint 2',
      backgroundColor: 'maroon',
      textColor: 'white',
      days: ["2023-12-24","2023-12-30"],
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'UI Desktop - Page đăng ký thành viên',
    schedule: [{
      id: 21,
      name: 'Tam',
      desc: '2023/12/24 - 2023/12/26',
      backgroundColor: 'maroon',
      textColor: 'white',
      days: ["2023-12-24","2023-12-26"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'API - Page đăng ký email',
    schedule: [{
      id: 21,
      name: 'Wilson',
      desc: '2023/12/27 - 2023/12/29',
      backgroundColor: 'maroon',
      textColor: 'white',
      days: ["2023-12-27","2023-12-29"]
    }],
  },
  {
    type: 'normal',
    color: '',
    name: 'Call API - Page đăng ký thành viên',
    schedule: [{
      id: 21,
      name: 'Tam',
      desc: '2023/12/28 - 2023/12/30',
      backgroundColor: 'maroon',
      textColor: 'white',
      days: ["2023-12-28","2023-12-30"]
    }],
  },
])
// Change Week Texts
// const cnWeekDays = ['日', '一', '二', '三', '四', '五', '六'];
let enWeekDays = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
let weekDays = document.getElementsByClassName('week');
let newWeekDays = [];
let dayInnerText;
const changeWeekDays = () => {
  for (let i = 0; i < weekDays.length; i++) {
    let newWeekDay = newWeekDays.push(weekDays[i]);
    let k = i%7;
    newWeekDay[i] = newWeekDay[k];
    dayInnerText = newWeekDay;
  }
  for (let j = 0; j < dayInnerText.length; j++) {
    switch (dayInnerText[j]) {
      case '日':
        dayInnerText[j].innerText = 'Sun';
        break;
      case '一':
        dayInnerText[j].innerText = 'Mon';
        break;
      case '二':
        dayInnerText[j].innerText = 'Tue';
        break;
      case '三':
        dayInnerText[j].innerText = 'Wed';
        break;
      case '四':
        dayInnerText[j].innerText = 'Thu';
        break;
      case '五':
        dayInnerText[j].innerText = 'Fri';
        break;
      case '六':
        dayInnerText[j].innerText = 'Sat';
    }
    console.log(dayInnerText);
  }
};
</script>
<style lang="scss">
.full-modal {
  .ant-modal {
    max-width: 100%;
    top: 0;
    padding-bottom: 0;
    margin: 0;
  }
  .ant-modal-content {
    display: flex;
    flex-direction: column;
    height: calc(100vh);
    min-height: calc(100vh);
    min-width: calc(100vw);
  }
  .ant-modal-body {
    flex: 1;
  }
}
.gantt {
 .work-desc {
   font-family: Alegreya,serif;
   font-size: 1.3rem!important;
   font-style: italic;
   text-align: center!important;
   padding-top: 12px;
 }
  .desc {
    visibility: hidden;
  }
  .month-item {
    background-color: rgba(24,144,255,0.05);

    .month {
      font-size: 1.2rem;
      font-family: Monospace,serif;
    }
  }
  .month, .day, .week {
    font-weight: 700;
    border: rgba(0,0,0,0.2) 1px solid!important;
  }
  .guide {
    border: rgba(0,0,0,0.2) 1px solid!important;
    background-color: rgba(253,181,21,0.05);
      width: 250px!important;
      .guide-name {
        border: rgba(0,0,0,0.2) 1px solid!important;
        font-weight: 500;
        font-family: Verdana,serif;
        font-size: 1rem;
        text-align: left;
      }
  }
  .minimize {
    max-width: 10px!important;
  }
}
</style>
