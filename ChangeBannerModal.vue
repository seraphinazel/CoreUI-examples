<template>
  <CModal :title="title" size="lg" color="info" :show.sync="showModal">
    <CCard>
      <CCardBody>
        <CRow>
          <CCol sm="4">
            <CCardText>Компания</CCardText>
            <CSelect
              placeholder="Выберите компанию"
              :options="companiesFiltered"
              :value.sync="bannerCompanyId"
              :isValid="bannerCompanyId !== null"
            />
          </CCol>
          <CCol sm="4">
            <CCardText>Название</CCardText>
            <CInput
              placeholder="Введите текст"
              v-model="bannerName"
              :isValid="bannerName !== null && bannerName !== ''"
            />
          </CCol>
          <CCol sm="4">
            <CCardText>Краткое описание</CCardText>
            <CInput
              placeholder="Введите текст"
              v-model="bannerText"
            />
          </CCol>
        </CRow>
        <CRow>
          <CCol sm="8">
            <CInputFile @change="onChange" />
          </CCol>
          <CCol sm="2">
            <CInputCheckbox
              label="Активировать баннер?"
              :checked.sync="bannerIsActive"
            />
          </CCol>
        </CRow>
      </CCardBody>
    </CCard>
    <template #footer>
      <CButton v-if="banner === 0" @click="createBanner" color="primary">Сохранить</CButton>
      <CButton v-else @click="editBanner" color="primary">Подтвердить изменения</CButton>
      <CButton color="secondary" @click="clearForm">Отменить</CButton>
    </template>
  </CModal>
</template>

<script>

export default{
  props: ['companies', 'userCompany'],
  data() {
    return {
      title: '',
      banner: 0,
      bannerName: null,
      bannerText: null,
      bannerImage: null,
      bannerCompanyId: null,
      bannerIsActive: false,
      showModal: false,
    };
  },
  computed: {
    companiesFiltered() {
      if (this.userCompany !== '') {
        return this.companies.filter(company => company.value === this.userCompany);
      } else {
        return this.companies;
      }
    },
  },
  methods: {
    show(item, title) {
      this.clearForm();
      this.showModal = true;
      this.title = title;
      if (item !== 0) {
        this.banner = Object.assign({}, item);
        this.getBaseItem();
      }
    },
    getBaseItem() {
      this.bannerName = this.banner.name;
      this.bannerCompanyId = this.banner.company_id; 
      this.bannerIsActive = this.banner.active;
      this.bannerText = this.banner.description;
    },
    onChange(FileList) {
      this.bannerImage = FileList[0];
    },
    createBanner() {
      this.$emit('isCreated', {
        name: this.bannerName,
        company_id: this.bannerCompanyId,
        image: this.bannerImage,
        active: this.bannerIsActive ? 1 : 0,
        description: this.bannerText,
      });
      this.showModal = false;
    },
    editBanner() {
      this.$emit('isEdited', {
        name: this.bannerName,
        company_id: this.bannerCompanyId,
        image: this.bannerImage,
        active: this.bannerIsActive ? 1 : 0,
        description: this.bannerText,
        itemId: this.banner.id,
      });
      this.showModal = false;
    },
    clearForm() {
      this.title = '';
      this.banner = 0;
      this.bannerName = null;
      this.bannerText = null;
      this.bannerImage = null;
      this.bannerCompanyId = null;
      this.bannerIsActive = false;
      this.showModal = false;
    },
  },
};
</script>
