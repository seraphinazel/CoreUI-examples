<template>
  <div>
    <ChangeHolidayModal
      v-if="userRole === 'admin' || userRole === 'company admin'"
      ref="changeHolidays"
      :months="monthes"
      :holidays="holidays"
      :companyId="companyId"
      @changed="getHolidays"
    />
    <CCard>
      <CCardHeader>
        <slot name="header">
          <CIcon name="cil-calendar"/> Календарь нерабочих дней
        </slot>
      </CCardHeader>
      <CCardBody>
        <CRow>
          <CCol sm="6">
            <CRow>
              <CCol>
                <CCardText>Чтобы сделать день нерабочим, кликните на соответствующую дату в календаре.</CCardText>
              </CCol>
            </CRow>
            <CRow>
              <CCol sm="6">
                <CInputCheckbox
                  label="Суббота - выходной"
                  :checked="satWeekend"
                  disabled
                />
              </CCol>
            </CRow>
            <CRow>
              <CCol sm="6">
                <CInputCheckbox
                  label="Воскресенье - выходной"
                  :checked="sunWeekend"
                  disabled
                />
              </CCol>
            </CRow>
          </CCol>
          <CCol sm="3">
            <CSelect
              label="Компания"
              placeholder="Выберите компанию"
              :options="companiesSelect"
              :value.sync="companyId"
              :disabled="editorCompanyId !== null"
            />
          </CCol>
          <CCol sm="3"></CCol>
        </CRow>
        <CSpinner
          color="primary"
          grow
          v-if="loading===true"
          class="load_spinner"
        />
        <CRow class="row-cols-1 row-cols-md-3">
          <CCol style="margin-top:20px; min-width:400px; max-width:400px;" v-for="item in fullYear" :key="item.index">
            <CCard class="h-100">
              <CCardHeader>
                <CRow alignHorizontal="center">
                  {{monthes[item.month]}} {{item.year}}
                </CRow>
              </CCardHeader>
              <CCardBody>
                <CRow>
                  <CCol
                    v-for="(d, index) in day" 
                    :key="d.index"
                    class="daycell"
                  >
                    {{d}}
                  </CCol>
                </CRow>
                <CRow v-for="(week, index) in calendar(item.year, item.month)" :key="week.index">
                  <CCol
                    v-for="(day, index) in week"
                    :style="{'color': day.weekend, 'background-color': day.current}"
                    :key="day.index"
                    class="daycell"
                    @click="userRole === 'admin' || userRole === 'company admin' ? $refs.changeHolidays.show(item.year, item.month, day) : ''"
                  >
                    {{ day.index }}
                  </CCol>
                </CRow>
              </CCardBody>
            </CCard>
          </CCol>
        </CRow>
      </CCardBody>
    </CCard>
  </div>
</template>

<script>
import axios from 'axios';
import ChangeHolidayModal from './ChangeHolidayModal.vue';

export default {
  components: {ChangeHolidayModal},
  data() {
    return {
      month: new Date().getMonth(),
			year: new Date().getFullYear(), 
			dFirstMonth: '1',
			day:["Пн", "Вт", "Ср", "Чт", "Пт", "Сб", "Вс"],
			monthes: ["Январь","Февраль","Март","Апрель","Май","Июнь","Июль","Август","Сентябрь","Октябрь","Ноябрь","Декабрь"],
      holidays: [], //список выходных дней из базы данных
      fullYear: [], //список месяцев на год вперед от текущей даты
      satWeekend: true,
      sunWeekend: true,
      loading: false,
      companies: [],
      companyId: 0,
      userRole: null
    };
  },
  computed: {
    companiesSelect() {
      return this.companies.map((item) => ({
        value: item.id ? item.id : 0,
        label: item.name ? item.name : 'Все',
      }));
    },
    holidaysFiltered() {
      return this.holidays.filter(item => (item.company_id === this.companyId || item.company_id == null));
    },
    editorCompanyId() {
        const id = localStorage.getItem('user_company_id');
        return id ? id : '';
    }
  },
  methods: {
    getHolidays() {
      this.loading = true;
      const url = this.$apiAdress + '/api/weekend'
        + '?company_id=' + this.companyId
        + '&token=' + localStorage.getItem('api_token');
      return axios.get(url).then(response => {
        this.holidays = response.data.data;
        this.loading = false;
      }).catch(error => {
        this.loading = false;
        this.requestError(error);
      });
    },
    getYearForward() {
      for (let i = 0; i <= 11; i++) {
        let newMonth = {
          month: new Date(this.year, this.month + i).getMonth(),
          year: new Date(this.year, this.month + i).getFullYear(),
        };
        this.fullYear.push(newMonth);
      };
    },
    getCompanies() {
      this.loading = true;
      const url = this.$apiAdress + '/api/company?token=' + localStorage.getItem('api_token');
      return axios.get(url).then(response => {
        this.companies = response.data.data;
        this.companies.push({value: 0, label: ''});
        this.loading = false;
      }).catch(error => {
        this.loading = false;
        this.requestError(error);
      });
    },
    calendar(year, month) {
      let days = [];
      let week = 0;
      days[week] = [];
      let dlast = new Date(year, month + 1, 0).getDate();
      for (let i = 1; i <= dlast; i++) {
        if (new Date(year, month, i).getDay() != this.dFirstMonth) {
          let a = {index: i};
          days[week].push(a);
          if (i == new Date().getDate() && year == new Date().getFullYear() && month == new Date().getMonth()) {
            a.current = '#747ae6';
          }
          if ((new Date(year, month, i).getDay() == 6 && this.satWeekend == true) || (new Date(year, month, i).getDay() == 0 && this.sunWeekend == true)){
            a.weekend = '#ff0000';
          }
          this.holidaysFiltered.forEach(holiday => {
            if (new Date(year, month, i).getDate() == holiday.day && new Date(year, month, i).getMonth() == holiday.month-1) {
              a.weekend = '#ff0000';
            }
          });
        } else {
          week++;
          days[week] = [];
          let a = {index:i};
          days[week].push(a);
          if ((i == new Date().getDate()) && (year == new Date().getFullYear()) && (month == new Date().getMonth())) {
            a.current = '#747ae6';
          }
          if ((new Date(year, month, i).getDay() == 6 && this.satWeekend == true) || (new Date(year, month, i).getDay() == 0 && this.sunWeekend == true)) {
            a.weekend = '#ff0000';
          }
          this.holidaysFiltered.forEach(holiday => {
            if (new Date(year, month, i).getDate() == holiday.day && new Date(year, month, i).getMonth() == holiday.month-1) {
              a.weekend = '#ff0000';
            }
          });
        }
      }
      if (days[0].length > 0) {
        for (let i = days[0].length; i < 7; i++) {
          days[0].unshift('');
        }
      }
      return days;
    },
    setWeekends() {
      if (this.companyId != 0) {
        let currentCompany = this.companies.find(item => item.id === this.companyId);
        let weekends = currentCompany.data.weekdays ? currentCompany.data.weekdays : null;
        this.satWeekend = this.sunWeekend = false;
        if (weekends !== null) {
          weekends.forEach(day => {
            if (day == 6) {
              this.satWeekend = true;
            } 
            if (day == 0) {
              this.sunWeekend = true;
            }
          });
        }
      } else {
        this.sunWeekend = true;
        this.satWeekend = true;
      }
    },
    /* updateCompanyWeekends() {
      if (this.companyId !== 0) {
        this.loading = true;
        const url = this.$apiAdress + '/api/company/' + this.companyId
            + '?company_id=' + this.editorCompanyId
            + '&token=' + localStorage.getItem('api_token');
        axios.put(url, {
          token: localStorage.getItem('api_token'),
          name: this.companies.find(item => item.id === this.companyId).name,
          data: {
            'weekdays': [
              this.satWeekend === true ? 6 : '',
              this.sunWeekend === true ? 0 : '']
          },
        }).then(() => {
          this.loading = false;
          this.flashSuccess('Успешно изменено');
          this.getCompanies();
        }).catch(error => {
          this.loading = false;
          this.requestError(error);
        });
      }
    }, */
  },
  watch: {
    companyId() {
      this.setWeekends();
    },
  },
  mounted() {
    this.getHolidays();
    this.getYearForward();
    this.getCompanies();
  },
  created() {
    this.companyId = this.editorCompanyId;
    this.userRole = localStorage.getItem('user_roles').split(',', 1);
  }
};
</script>

<style scoped>
.daycell {
  padding: 5px !important;
  max-width: 50px !important;
  min-width: 50px !important;
  text-align: center;
  cursor: pointer;
}
</style>