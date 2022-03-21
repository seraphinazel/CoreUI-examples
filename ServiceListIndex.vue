<template>
  <div>
    <ChangeServiceModal
      ref="changeService"
      :groups="serviceGroups"
      :sets="sets"
      :companies="companies"
      @isEdited="editService"
      @isCreated="createService"
    />
    <ChangeServiceGroupModal
      ref="createServiceGroup"
      :groups="serviceGroups"
      :companies="companies"
      :editorCompanyId="editorCompanyId"
      @isCreated="createServiceGroup"
    />
    <DeleteConfirmModal
      ref="deleteServiceElement"
      title="Удалить элемент"
      @deleted="deleteService"
    />
    <CCardGroup>
      <transition name="fade">
        <CCard>
          <CCardHeader>
            <slot name="header">
              <CIcon name="cil-spa"/> Группы и подгруппы услуг
            </slot>
          </CCardHeader>
          <CCardBody>
            <CSpinner
              color="primary"
              grow
              v-if="loading===true"
              class="load_spinner"
            />
            <ServiceGroupsInner 
              :groups="serviceGroups"
              :allGroups="serviceGroups"
              :companies="companies"
              @services="showServices"
              @groupDeleted="deleteGroup"
              @editGroup="editServiceGroup"
            />
          </CCardBody>
          <CCardFooter>
            <CButton
              v-if="userRole === 'admin' || userRole === 'company admin'"
              block
              color="primary"
              variant="outline"
              @click="$refs.createServiceGroup.show(0, 'Создание новой группы услуг')"
            >
              Создать новую группу
            </CButton>
          </CCardFooter>
        </CCard>
      </transition>
      <CCard>
        <CCardHeader>
          <slot name="header">
            <CIcon name="cil-leaf"/> Услуги
          </slot>
        </CCardHeader>
        <CCardBody>
          <CListGroup>
            <CListGroupItem v-for="service in shownGroup.services" :key="service.id">
              <CRow>
                <CCol sm="4">
                  {{service.name}}
                </CCol>
                <CCol sm="3" v-if="service.set !== null">
                  ({{service.set.id}}. {{service.set.name}})
                </CCol>
                <CCol sm="3">
                  {{_cformat(service.min_price)}} - {{_cformat(service.max_price)}}
                </CCol>
                <CCol sm="1">
                  <CButton
                    v-if="userRole === 'admin' || userRole === 'company admin'"
                    size="sm"
                    color="primary"
                    variant="ghost"
                    @click="$refs.changeService.show(service, 'Редактирование услуги')"
                  >
                    <CIcon name="cil-pencil"/>
                  </CButton>
                </CCol>
                <CCol sm="1">
                  <CButton
                    v-if="userRole === 'admin' || userRole === 'company admin'"
                    size="sm"
                    color="danger"
                    variant="ghost"
                    @click="confirmDelete(service)"
                  >
                    <CIcon name="cil-x-circle"/>
                  </CButton>
                </CCol>
              </CRow>
            </CListGroupItem>
          </CListGroup>
        </CCardBody>
        <CCardFooter>
          <CButton
            v-if="userRole === 'admin' || userRole === 'company admin'"
            block
            color="primary"
            variant="outline"
            @click="$refs.changeService.show(0, 'Создать новую услугу')"
          >
            Создать новую услугу
          </CButton>
        </CCardFooter>
      </CCard>
    </CCardGroup>
  </div>
</template>

<script>
import ChangeServiceModal from './ChangeServiceModal';
import ChangeServiceGroupModal from './ChangeServiceGroupModal';
import DeleteConfirmModal from '../../common/DeleteConfirmModal';
import ServiceGroupsInner from './ServiceGroupsInner';

export default {
    components: {
        ChangeServiceModal,
        DeleteConfirmModal,
        ServiceGroupsInner,
        ChangeServiceGroupModal
    },
    data() {
        return {
            loading: false,
            serviceGroups: [],
            shownGroup: {},
            companies: [],
            sets: [],
            userRole: null
        };
    },
    methods: {
        getServiceGroups() {
            this.loading = true;
            const url = this.$apiAdress + '/api/service_group' 
                + '?token=' + localStorage.getItem('api_token') 
                + '&company_id=' + this.editorCompanyId;
            return axios.get(url).then(response => {
                this.serviceGroups = response.data.data;
                this.loading = false;
            }).catch(error => {
                this.loading = false;
                this.requestError(error);
            });
        },
        getCompanies() {
            this.loading = true;
            const url = this.$apiAdress + '/api/company'
                + '?company_id=' + this.editorCompanyId
                + '&token=' + localStorage.getItem('api_token');
            return axios.get(url).then(response => {
                this.companies = response.data.data;
                this.loading = false;
            }).catch(error => {
                this.loading = false;
                this.requestError(error);
            });
        },
        getSets() {
            this.loading = true;
            const url = this.$apiAdress + '/api/set'
                + '?company_id=' + this.editorCompanyId
                + '&token=' + localStorage.getItem('api_token');
            return axios.get(url).then(response => {
                this.sets = response.data.data;
                this.loading = false;
            }).catch(error => {
                this.loading = false;
                this.requestError(error);
            });
        },
        showServices(data) {
          this.shownGroup = data;
        },
        deleteGroup(data) {
            this.loading = true;
            const url = this.$apiAdress + '/api/service_group/' + data.itemId 
                + '?token=' + localStorage.getItem("api_token");
            return axios.delete(url).then(() => {
                this.loading = false;
                this.shownGroup = {};
                this.flashSuccess('Успешно удалено');
                this.getServiceGroups();
            }).catch((error) => {
                this.loading = false;
                this.requestError(error);
            });
        },
        deleteService(data) {
            this.loading = true;
            const url = this.$apiAdress + '/api/service/' + data.itemId 
                + '?token=' + localStorage.getItem("api_token");
            return axios.delete(url).then(() => {
                this.loading = false;
                this.shownGroup = {};
                this.flashSuccess('Успешно удалено');
                this.getServiceGroups();
            }).catch((error) => {
                this.loading = false;
                this.requestError(error);
            });
        },
        editService(data) {
          this.loading = true;
          return axios.put(this.$apiAdress + '/api/service/' + data.itemId, {
                token: localStorage.getItem("api_token"),
                company_id: data.company_id,
                min_price: data.min_price,
                max_price: data.max_price,
                name: data.name,
                service_group_id: data.service_group_id,
                set_id: data.set_id,
            }).then(() => {
                this.loading = false;
                this.shownGroup = {};
                this.flashSuccess('Успешно изменено');
                this.getServiceGroups();
            }).catch((error) => {
                this.loading = false;
                this.requestError(error);
            });
        },
        createService(data) {
            this.loading = true;
            return axios.post(this.$apiAdress + '/api/service', {
                token: localStorage.getItem("api_token"),
                company_id: data.company_id,
                min_price: data.min_price,
                max_price: data.max_price,
                name: data.name,
                service_group_id: data.service_group_id,
                set_id: data.set_id,
            }).then(() => {
                this.loading = false;
                this.shownGroup = {};
                this.flashSuccess('Успешно добавлено');
                this.getServiceGroups();
            }).catch((error) => {
                this.loading = false;
                this.requestError(error);
            });
        },
        createServiceGroup(data) {
            this.loading = true;
            return axios.post(this.$apiAdress + '/api/service_group', {
                token: localStorage.getItem("api_token"),
                company_id: data.company_id,
                name: data.name,
                service_group_id: data.service_group_id,
            }).then(() => {
                this.loading = false;
                this.flashSuccess('Успешно добавлено');
                this.getServiceGroups();
            }).catch((error) => {
                this.loading = false;
                this.requestError(error);
            });
        },
        editServiceGroup(data) {
          this.loading = true;
          return axios.put(this.$apiAdress + '/api/service_group/'+ data.itemId, {
                token: localStorage.getItem("api_token"),
                company_id: data.company_id,
                name: data.name,
                service_group_id: data.service_group_id,
          }).then(() => {
                this.loading = false;
                this.flashSuccess('Успешно изменено');
                this.getServiceGroups();
          }).catch((error) => {
                this.loading = false;
                this.requestError(error);
          });
        },
        confirmDelete(item) {
            this.$refs.deleteServiceElement.show(item);
        }
    },
    mounted() {
        this.getServiceGroups();
        this.getCompanies();
        this.getSets();
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
