<template>
  <CModal :title="title" size="lg" color="info" :show.sync="showModal">
    <CCard>
      <CCardBody>
        <CRow>
          <CCol sm="4">
            <CCardText>Компания</CCardText>
            <CSelect
              placeholder="Выберите компанию"
              :options="companiesSelect"
              :value.sync="serviceGroupCompanyId"
              :isValid="serviceGroupCompanyId > 0"
              :disabled="userCompanyId !== ''"
            />
          </CCol>
          <CCol sm="4">
            <CCardText>Название</CCardText>
            <CInput
              placeholder="Введите название"
              v-model="serviceGroupName"
              :isValid="serviceGroupName !== null && serviceGroupName !== ''"
            />
          </CCol>
          <CCol sm="4" v-if="serviceGroupCompanyId > 0">
            <CCardText>Родительская группа</CCardText>
            <treeselect 
              placeholder="Выберите родительскую группу услуг" 
              :value.sync="serviceGroupId"
              v-model="serviceGroupId"
              :options="filteredServiceGroups"
              :searchable="true"
              :clearable="true"
              :close-on-select="true"
              :open-on-click="true"
              :default-expand-level="Infinity"
            />
          </CCol>
        </CRow>
      </CCardBody>
    </CCard>
    <template #footer>
      <CButton v-if="serviceGroup === 0" @click="createServiceGroup" color="primary">Сохранить</CButton>
      <CButton v-else @click="editServiceGroup" color="primary">Подтвердить изменения</CButton>
      <CButton color="secondary" @click="clearForm">Отменить</CButton>
    </template>
  </CModal>
</template>

<script>
import Treeselect from '@riophae/vue-treeselect';
import '@riophae/vue-treeselect/dist/vue-treeselect.css';

export default {
  components: {Treeselect},
  props: ['groups', 'companies', 'userCompanyId'],
  data() {
    return {
      title: '',
      serviceGroup: 0,
      serviceGroupName: null,
      serviceGroupId: 0,
      serviceGroupCompanyId: null,
      showModal: false,
      filteredServiceGroups: [],
    };
  },
  computed: {
    companiesSelect() {
      return this.companies.map(company => ({
        value: company.id,
        label: company.name,
      }));
    },
  },
  methods: {
    show(item, title) {
      this.clearForm();
      this.showModal = true;
      this.title = title;
      this.serviceGroupCompanyId = this.userCompanyId;
      if (item !== 0) {
        let itemCopy = Object.assign({}, item);
        this.serviceGroup = itemCopy;
        this.getBaseItem();
      }
    },
    groupsNormalized(groups) {
      let filtered = groups.filter(group => group.company_id === this.serviceGroupCompanyId);
      return filtered.map((item) => {
        if (item.child_groups.length > 0) {
          return {
            id: item.id,
            label: item.name,
            children: this.groupsNormalized(item.child_groups)
          };
        } else {
          return {
            id: item.id,
            label: item.name,
          };
        }
      });
    },
    getBaseItem() {
      this.serviceGroupName = this.serviceGroup.name;
      this.serviceGroupId = this.serviceGroup.service_group_id ? this.serviceGroup.service_group_id : null;
      this.serviceGroupCompanyId = this.serviceGroup.company_id;
    },
    createServiceGroup() {
      this.$emit('isCreated', {
        name: this.serviceGroupName,
        service_group_id: this.serviceGroupId !== null ? this.serviceGroupId : null,
        company_id: this.serviceGroupCompanyId,
      });
      this.showModal = false;
    },
    editServiceGroup() {
      this.$emit('isEdited', {
        name: this.serviceGroupName,
        service_group_id: this.serviceGroupId !== null ? this.serviceGroupId : null,
        company_id: this.serviceGroupCompanyId,
        itemId: this.serviceGroup.id,
      });
      this.showModal = false;
    },
    clearForm() {
      this.title = '';
      this.serviceGroup = 0;
      this.serviceGroupName = null;
      this.serviceGroupId = null;
      this.serviceGroupCompanyId = null;
      this.showModal = false;
      this.filteredServiceGroups = [];
    },
  },
  watch: {
    serviceGroupCompanyId() {
      this.filteredServiceGroups = this.groupsNormalized(this.groups);
    },
  },
};
</script>
