<template>
<CListGroup>
  <DeleteConfirmModal
    ref="deleteServiceGroup"
    title="Удалить группу"
    @deleted="deleteThisGroup"
  />
  <ChangeServiceGroupModal
     ref="editServiceGroup"
    :groups="allTreeGroups"
    :companies="companies"
    @isEdited="editThisGroup"
  />
  <CListGroupItem v-for="group in groups" :key="group.id" href="#" color="light">
      <CRow @click="showServices(group)">
        <CCol sm="6">
          <CCardText>{{group.name}}</CCardText>
        </CCol>
        <CCol sm="2">
          <CLink class="card-header-action btn-minimize" @click="toggleDetails(group)">
            <CIcon :name="`cil-chevron-${group.showDetails ? 'top' : 'bottom'}`"/>
          </CLink>
        </CCol>
        <CCol sm="2">
          <CButton
            v-if="(userRole === 'admin' || userRole === 'company admin') && groupEditable(group)"
            size="sm"
            color="primary"
            variant="ghost"
            @click="$refs.editServiceGroup.show(group, 'Редактирование группы услуг')"
          >
            <CIcon name="cil-pencil"/>
          </CButton>
        </CCol>
        <CCol sm="2">
          <CButton
            v-if="userRole === 'admin' || userRole === 'company admin'"
            size="sm"
            color="danger"
            variant="ghost"
            @click="confirmDelete(group)"
          >
            <CIcon name="cil-x-circle"/>
          </CButton>
        </CCol>
      </CRow>
    <CCollapse :show="group.showDetails===true" :duration="400">
      <ServiceGroupsInner
        :allGroups = "allTreeGroups"
        :groups="group.child_groups"
        :companies="companies"
        @services="showServices"
        @groupDeleted="deleteThisGroup"
        @editGroup="editThisGroup"
      />
    </CCollapse>
  </CListGroupItem>
</CListGroup>
</template>

<script>
import ServiceGroupsInner from './ServiceGroupsInner';
import DeleteConfirmModal from '../../common/DeleteConfirmModal';
import ChangeServiceGroupModal from './ChangeServiceGroupModal';

export default {
  name: 'ServiceGroupsInner',
  components: {ServiceGroupsInner, DeleteConfirmModal, ChangeServiceGroupModal},
  props: ['groups', 'companies', 'allGroups'],
  data() {
    return {
      userRole: null,
    };
  },
  computed: {
    allTreeGroups() {
      return this.allGroups;
    },
  },
  methods: {
    showServices(data) {
      this.$emit('services', data);
    },
    toggleDetails(group) {
      if (group.showDetails === undefined) {
        this.$set(group, 'showDetails', true);
      } else {
        group.showDetails = !group.showDetails;
      }
    },
    deleteThisGroup(data) {
      this.$emit('groupDeleted', data);
    },
    confirmDelete(item) {
      this.$refs.deleteServiceGroup.show(item);
    },
    editThisGroup(data) {
      this.$emit('editGroup', data);
    },
    groupEditable(group) {
        return group.services.findIndex(s => s.used) === -1;
    }
  },
  created() {
    this.userRole = localStorage.getItem('user_roles').split(',', 1);
  },
};
</script>
