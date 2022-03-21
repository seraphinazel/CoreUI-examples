<template>
  <CModal title="Поставить/снять отметку о нерабочем дне" color="info" :show.sync="showModal">
    <CCard>
      <CCardHeader>{{date ? date.index : '' + ' '}} {{months[month]}}</CCardHeader>
      <CCardBody>
        <CRow>
          <CCol sm="6">
            <CInputCheckbox
              label="Нерабочий день"
              :checked.sync="isHoliday"
            />
          </CCol>
          <CCol sm="6">
            <CInputCheckbox
              label="Ежегодный"
              :checked.sync="annually"
            />
          </CCol>
        </CRow>
      </CCardBody>
    </CCard>
    <template #footer>
      <CButton color="primary" @click="saveChanges">Сохранить изменения</CButton>
      <CButton color="secondary" @click="clearForm">Закрыть окно</CButton>
    </template>
  </CModal>
</template>

<script>
import axios from 'axios';

export default{
  props: ['months', 'holidays', 'companyId'],
  data() {
    return {
      year: null,
      month: null,
      date: {},
      isHoliday: null,
      annually: false,
      showModal: false,
      holidayId: null,
    };
  },
  methods: {
    show(year, month, day) {
      this.showModal = true;
      this.date = day;
      this.month = month;
      this.year = year;
      let holiday = this.holidays.find(item => item.day == day.index && item.month == month+1);
      if (holiday !== undefined) {
        this.isHoliday = true;
        this.holidayId = holiday.id;
      }
    },
    clearForm() {
      this.showModal = false;
      this.date = {};
      this.month = null;
      this.isHoliday = null;
      this.annually = false;
      this.holidayId = null;
    },
    saveChanges() {
      if (this.isHoliday == true) {
        this.createHoliday();
      } else if (this.isHoliday == false) {
        this.deleteHoliday();
      }
    },
    createHoliday() {
      let newDate = new Date(Date.UTC(this.year, this.month, this.date.index, 0, 0, 0));
      newDate = Math.floor(Date.parse(newDate) / 1000);
      if (this.companyId !== 0) {
        axios.post(this.$apiAdress + '/api/weekend', {
          token: localStorage.getItem('api_token'),
          company_id: this.companyId,
          annually: this.annually,
          date: newDate
        }).then(() => {
          this.$emit('changed');
          this.clearForm();
          this.flashSuccess('Успешно добавлено');
        }).catch(error => {
          this.requestError(error);
        });
      } else {
        axios.post(this.$apiAdress + '/api/weekend', {
          token: localStorage.getItem('api_token'),
          annually: this.annually,
          date: newDate
        }).then(() => {
          this.$emit('changed');
          this.clearForm();
          this.flashSuccess('Успешно добавлено');
        }).catch(error => {
          this.requestError(error);
        });
      }
    },
    deleteHoliday() {
      const url = this.$apiAdress 
      + '/api/weekend/' 
      + this.holidayId 
      + '?token=' 
      + localStorage.getItem("api_token");
      return axios.delete(url)
      .then(() => {
        this.$emit('changed');
        this.clearForm();
      })
      .catch((error) => {
        this.requestError(error);
      });
    },
  },
};
</script>
