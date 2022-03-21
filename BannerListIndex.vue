<template>
<div>
  <DeleteConfirmModal
    ref="deleteBanner"
    title="Удалить баннер"
    @deleted="deleteBanner"
  />

  <ChangeBannerModal
    ref="changeBanner"
    :companies="companies"
    :userCompany="editorCompanyId"
    @isCreated="createBanner"
    @isEdited="editBanner"
  />
  <CCol>
    <CRow>
      <CCol md="3">
        <CButton
          v-if="userRole === 'admin' || userRole === 'company admin'"
          color="primary"
          variant="outline"
          size="lg"
          @click="$refs.changeBanner.show(0, 'Создание нового баннера')"
        >
          Создать баннер
        </CButton>
      </CCol>
    </CRow>
    <CRow>
      <CSpinner
        color="primary"
        grow
        v-show="loading===true"
        class="load_spinner"
      />
    </CRow>
    <CRow class="row-cols-1 row-cols-md-2">
      <CCol style="margin-top:20px;" v-for="banner in banners" :key="banner.id">
        <CCard class="h-100" :borderColor="`${banner.active ? 'success' : 'secondary'}`" :color="`${banner.active ? '' : 'light'}`">
          <CCardBody>
          <CRow>
            <CCol sm="10">
            <CCardTitle v-if="userRole === 'admin'">{{getCompanyName(banner.company_id)}}</CCardTitle>
            <CCardImg
              :src="banner.image"
              :alt="banner.name"
            />
            <CCardTitle>{{banner.description !== null ? banner.description : ''}}</CCardTitle>
          
            </CCol>
            <CCol sm="2">

            <CButtonGroup vertical v-if="userRole === 'admin' || userRole === 'company admin'">
              <CButton
                :color="`${banner.active ? 'secondary' : 'success'}`"
                variant='outline'
                @click="activateBanner(banner)"
                v-c-tooltip="'Включить/выключить баннер'"
              >
                <CIcon :name="`${ banner.active ? 'cil-x-circle' : 'cil-check-circle'}`"/>
              </CButton>
              <CButton
                color="primary"
                variant='outline'
                @click="$refs.changeBanner.show(banner, 'Редактирование баннера')"
                v-c-tooltip="'Редактировать баннер'"
              >
                <CIcon name="cil-pencil"/>
              </CButton>
              <CButton
                color="danger"
                variant='outline'
                v-c-tooltip="'Удалить баннер'"
                @click="$refs.deleteBanner.show(banner)"
                >
                <CIcon name="cil-x-circle"/>
              </CButton>
            </CButtonGroup>

            </CCol>
          </CRow>
          </CCardBody>
          
        </CCard>
      </CCol>
    </CRow>
  </CCol>
</div>
</template>

<script>
import DeleteConfirmModal from '../../common/DeleteConfirmModal';
import ChangeBannerModal from './ChangeBannerModal';

export default {
  data() {
    return {
      banners: [],
      companies: [],
      loading: false,
      userRole: null
    };
  },
  components: {DeleteConfirmModal, ChangeBannerModal},
  methods: {
    getBanners() {
      this.loading = true;
      const url = this.$apiAdress + '/api/banner?token=' + localStorage.getItem('api_token') + "&company_id=" + this.editorCompanyId;
      return axios.get(url).then(response => {
        this.banners = response.data.data;
        this.loading = false;
      }).catch(error => {
        this.loading = false;
        this.requestError(error);
      });
    },
    getCompanies() {
      this.loading = true;
      const url = this.$apiAdress + '/api/company?token=' + localStorage.getItem('api_token');
      return axios.get(url).then(response => {
        this.companies = response.data.data.map((item) => ({
          value: item.id,
          label: item.name,
        }));
        this.loading = false;
      }).catch(error => {
        this.loading = false;
        this.requestError(error);
      });
    },
    deleteBanner(data) {
      this.loading = true;
      const url = this.$apiAdress + '/api/banner/' + data.itemId + '?token=' + localStorage.getItem("api_token");
      return axios.delete(url).then(() => {
        this.loading = false;
        this.flashSuccess('Успешно удалено');
        this.getBanners();
      }).catch((error) => {
        this.requestError(error);
        this.getBanners();
      });
    },
    createBanner(data) {
      this.loading = true;
      let bannerData = new FormData();
      bannerData.append('file', data.image);
      bannerData.append('name', data.name);
      bannerData.append('active', data.active);
      bannerData.append('company_id', data.company_id);
      data.description !== null ? bannerData.append('description', data.description) : '';
      const url = this.$apiAdress + '/api/banner?token='+ localStorage.getItem('api_token');
      axios.post(url, bannerData, {
        headers: {'content-type': 'multipart/form-data'}
      }).then(res => {
        this.flashSuccess('Создан баннер ID:' + res.data.id);
        this.getBanners();
        this.loading = false;
      }).catch(err => {
        this.requestError(err);
        this.getBanners();
      });
    },
    editBanner(data) {
      this.loading = true;
      let bannerData = new FormData();
      data.image !== null ? bannerData.append('file', data.image) : '';
      bannerData.append('name', data.name);
      bannerData.append('active', data.active);
      bannerData.append('company_id', data.company_id);
      data.description !== null ? bannerData.append('description', data.description) : '';
      bannerData.append('_method', 'PUT');
      const url = this.$apiAdress + '/api/banner/' + data.itemId + '?token=' + localStorage.getItem("api_token");
      axios.post(url, bannerData, {
        headers: {'content-type': 'multipart/form-data'}
      }).then(res => {
        this.flashSuccess('Баннер №' + res.data.data.id + ' успешно изменён!');
        this.getBanners();
        this.loading = false;
      }).catch(err => {
        this.requestError(err);
        this.getBanners();
      });
    },
    activateBanner(banner) {
      banner.active === true ? banner.active = 0 : banner.active = 1;
      banner.itemId = banner.id;
      banner.image = null;
      this.editBanner(banner);
    },
    getCompanyName(id) {
      let company = this.companies.find(item => item.value == id);
      return company.label;
    }
  },
  mounted() {
    this.getCompanies();
    this.getBanners();
  },
  created() {
    this.userRole = localStorage.getItem('user_roles').split(',', 1);
  },
  computed: {
    editorCompanyId() {
        const id = localStorage.getItem('user_company_id');
        return id ? id : '';
    }
  }
};
</script>
