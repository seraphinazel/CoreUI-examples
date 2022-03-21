<template>
  <CModal :title="title" size="lg" color="info" :show.sync="showModal">
    <CCard>
      <CCardBody>
        <CRow>
          <CCol sm="6">
            <CInput
              label="Название"
              placeholder="Введите название"
              v-model.trim="serviceName"
              :disabled="serviceDisabled"
              :isValid="serviceName !== '' && serviceName !== ''"
            />
          </CCol>
          <CCol sm="3">
            <CInput
              label="Мин.цена"
              placeholder="Введите число"
              v-model.number="serviceMinPrice"
              :isValid="serviceMinPrice !== '' && serviceMinPrice >= 0"
            />
          </CCol>
          <CCol sm="3">
            <CInput
              label="Макс.цена"
              placeholder="Введите число"
              v-model.number="serviceMaxPrice"
              :isValid="serviceMaxPrice !== '' && serviceMaxPrice >= serviceMinPrice"
            />
          </CCol>
        </CRow>
        <CRow>
          <CCol sm="6">
            <CCardText>Группа услуг</CCardText>
            <treeselect
              placeholder="Выберите группу услуг" 
              v-model="serviceGroupId" 
              :options="allServiceGroups"
              :searchable="true"
              :clearable="true"
              :close-on-select="true"
              :open-on-click="true"
              :default-expand-level="Infinity"
              :disabled="serviceDisabled"
            />
                          <!--:isValid="serviceGroupId !== null" !!! -->
          </CCol>
          <CCol sm="6" v-if="serviceSetId !== 'no_set'">
            <CCardText>Комплект</CCardText>
            <CSelect
              placeholder="Выберите комплект"
              :options="setsSelect"
              :value.sync="serviceSetId"
              :disabled="serviceDisabled"
            />
          </CCol>
        </CRow>
      </CCardBody>
    </CCard>
    <template #footer>
      <CButton  color="primary" @click="service ? editService() : createService()" :disabled="!dataValid">Сохранить</CButton>
      <CButton color="secondary" @click="clearForm">Отменить</CButton>
    </template>
  </CModal>
</template>

<script>
import Treeselect from '@riophae/vue-treeselect';
import '@riophae/vue-treeselect/dist/vue-treeselect.css';

export default {
  components: {Treeselect},
  props: ['groups', 'sets'],
  data() {
    return {
      title: '',
      service: 0,
      serviceName: '',
      serviceMinPrice: '',
      serviceMaxPrice: '',
      serviceGroupId: null,
      serviceSetId: null,
      serviceCompanyId: null,
      showModal: false,
      allServiceGroups: [],
    };
  },
  computed: {
    setsSelect() {
      return this.sets.map(set => ({
        value: set.id,
        label: set.name
      }));
    },
    serviceDisabled() {
        return this.service !== 0 && this.service.used;
    },
    dataValid() {
        return this.serviceName !== '' 
            && this.serviceMinPrice !== ''
            && this.serviceMaxPrice !== ''
            && this.serviceGroupId !== null;
    }
  },
  methods: {
    show(item, title) {
      this.clearForm();
      this.showModal = true;
      this.title = title;
      this.allServiceGroups = this.groupsNormalized(this.groups);
      if (item !== 0) {
        this.service = Object.assign({}, item);
        this.getBaseItem();
      }
    },
    groupsNormalized(groups) {
      return groups.map((item) => {
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
      this.serviceName = this.service.name;
      this.serviceMinPrice = this.service.min_price;
      this.serviceMaxPrice = this.service.max_price;
      this.serviceGroupId = this.service.service_group_id;
      this.serviceSetId = this.service.set !== null ? this.service.set.id : 'no_set';
      this.serviceCompanyId = this.service.company_id;
    },
    getCompanyIdFromGroup(groups) {
      let group = groups.find(item => item.id === this.serviceGroupId);
      if (group === undefined) {
        groups.forEach(elem => {
          this.getCompanyIdFromGroup(elem.child_groups);
        });
      } else {
        this.serviceCompanyId = group.company_id;
      }
    },
    editService() {
      this.getCompanyIdFromGroup(this.groups);
      this.$emit('isEdited', {
        name: this.serviceName,
        min_price: this.serviceMinPrice,
        max_price: this.serviceMaxPrice,
        service_group_id: this.serviceGroupId,
        set_id: this.serviceSetId,
        company_id: this.serviceCompanyId,
        itemId: this.service.id,
      });
      this.showModal = false;
    },
    createService() {
      this.getCompanyIdFromGroup(this.groups);
      this.$emit('isCreated', {
        name: this.serviceName,
        min_price: this.serviceMinPrice,
        max_price: this.serviceMaxPrice,
        service_group_id: this.serviceGroupId,
        set_id: this.serviceSetId,
        company_id: this.serviceCompanyId,
      });
      this.showModal = false;
    },
    clearForm() {
      this.title = '';
      this.service = 0;
      this.serviceName = '';
      this.serviceMinPrice = '';
      this.serviceMaxPrice = '';
      this.serviceGroupId = null;
      this.serviceSetId = null;
      this.serviceCompanyId = null;
      this.showModal = false;
      this.allServiceGroups = [];
    },
  },
};
</script>
