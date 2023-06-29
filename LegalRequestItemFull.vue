/*
*
* Сделано в студии AppFox
*
*/
<template>
  <div>
    <div class="h1 mb-16">
      {{ course.title }}
    </div>
    <button
      :class="$style.RequestAddStudent"
      class="btn btn-border-green btn-hover btn-full-width"
      @click="$root.$emit(`add-students-modal:modal:open`)"
    >
      Добавить студента
    </button>
    <Modal
      :id="`add-students-modal`"
      size="lg"
      header-margin="small"
    >
      <template #title>
        Добавить обучающихся
      </template>

      <template #default>
        <AddStudentForm
          :course-id="course.id"
          :students="studentsToAdd"
          :selected="selectedStudents"
          @save="selectStudent"
        />
      </template>
    </Modal>
    <LegalRequestItem
      :data="course"
      :full="true"
      :full-icon="true"
    >
      <div
        :class="$style.RequestSubInfoContent"
        class="d-flex flex-wrap justify-content-between pos-rel"
      >
        <div
          :class="$style.RequestSubInfoContentItem"
          class="mb-16"
        >
          <div class="gray-label">
            Форма обучения:
          </div>
          <div>
            {{ course.study_form }}
          </div>
        </div>

        <div
          :class="$style.RequestSubInfoContentItem"
          class="mb-16"
        >
          <div class="gray-label">
            Место проведения:
          </div>
          <div>
            {{ course.locations[0] }}
            <button
              class="reset-button blue-dashed-link"
              @click="$root.$emit('schedule-places:modal:open')"
            >
              Посмотреть все адреса
            </button>
          </div>
        </div>

        <div :class="$style.RequestSubInfoContentItem">
          <div class="gray-label">
            Длительность обучения:
          </div>
          <div>
            {{ `${Math.trunc(course.duration.days / 7)} неделя (${course.duration.hours}ч)` }}
          </div>
        </div>

        <div :class="$style.RequestSubInfoContentItem">
          <div class="gray-label">
            Получаемый документ:
          </div>
          <div>
            {{ receivedDocuments(course.received_documents) }}
          </div>
        </div>
      </div>

      <div :class="$style.RequestStudentsTable">
        <Table
          template="default"
          :borders="false"
        >
          <TableHead>
            <TableRow>
              <TableCol>
                <div class="col-name">
                  <TableFilter
                    :filters="filtersForTable('full_name')"
                    name="full_name"
                    @filter="filterTable('full_name', $event)"
                    @search="searchInCoursesTable('full_name', $event)"
                    @sort-asc="sortTable('full_name', 'asc')"
                    @sort-desc="sortTable('full_name', 'desc')"
                  >
                    <span class="gray-color">ФИО</span>
                  </TableFilter>
                </div>
              </TableCol>

              <TableCol>
                <div class="col-dates">
                  <TableFilter
                    :filters="[`${course.start_edu} - ${course.end_edu}`]"
                    name="name"
                  >
                    <span class="gray-color">Даты обучения:</span>
                  </TableFilter>
                </div>
              </TableCol>

              <TableCol>
                <div class="col-status">
                  <TableFilter
                    :filters="['В порядке', 'На проверке', 'Не все доки']"
                    name="status"
                    @filter="filterTable('status_documents', $event)"
                    @search="searchInCoursesTable('status_documents', $event)"
                    @sort-asc="sortTable('status_documents', 'asc')"
                    @sort-desc="sortTable('status_documents', 'desc')"
                  >
                    <span class="gray-color">Статус документов</span>
                  </TableFilter>
                </div>
              </TableCol>

              <TableCol>
                <div class="col-action gray-color">
                  Действие
                </div>
              </TableCol>
            </TableRow>
          </TableHead>

          <TableBody>
            <TableRow
              v-for="student in filteredStudents"
              :key="student.profile_id"
            >
              <TableCol>
                <div class="col-name">
                  {{ student.full_name }}
                </div>
              </TableCol>

              <TableCol>
                <div class="col-dates">
                  {{ course.start_edu }} - {{ course.end_edu }}
                </div>
              </TableCol>

              <TableCol>
                <div class="col-status d-flex">
                  <span class="mr-16">({{ student.status_documents.count_accept }}/{{ student.status_documents.count_all }})</span>
                  <DocumentAdmissions
                    v-if="getDocumentStatus(student.admissions) === 'error'"
                    :admissions="getErrorDocuments(student.admissions)"
                    :other-doc-types="otherDocTypes"
                    :company-info="companyInfo"
                    :order-id="businessOrderId"
                    :profile-id="student.profile_id"
                    @submit="submitAdmissionsHandler"
                  />
                </div>
              </TableCol>

              <TableCol>
                <div class="col-action d-flex">
                  <nuxt-link
                    to="/profile/my-requests/request-1234-legal/chat/1"
                    class="d-flex mr-16"
                  >
                    <Icon
                      name="message"
                      color="dark-gray"
                      hover="green"
                    />
                  </nuxt-link>

                  <button
                    class="reset-button d-flex"
                    @click="removeStudents(student)"
                  >
                    <Icon
                      name="trash"
                      color="dark-gray"
                      hover="red"
                    />
                  </button>
                </div>
              </TableCol>
            </TableRow>
          </TableBody>
        </Table>
      </div>
    </LegalRequestItem>

    <Modal
      id="schedule-places"
      size="md"
    >
      <template #title>
        Адреса проведения
      </template>

      <CourseSchedule :addresses="course.locations" />
    </Modal>
  </div>
</template>

<script lang="ts">
import Component, { mixins } from 'vue-class-component';
import { getDocumentName } from '@/helpers/documents';
import { mapActions, mapState } from 'vuex';
import { IReceiveDocuments, IRequestStudent, ISingleRequestCourse, StudentRequestFields } from '~/types/interfaces/Request.interface';
import { IStudent } from '~/types/interfaces/User.interface';
import { IBusinessProfileItem } from '~/types/interfaces/Profile.interface';
import AdmissionsMixin from '~/mixins/Orders/AdmissionsMixin';
import { IOtherDocType } from '~/types/interfaces/Documents.interface';

@Component({
  computed: {
    ...mapState('request', ['course', 'students']),
    ...mapState('user', ['activeProfile'])
  },
  methods: {
    ...mapActions('request', ['removeListener', 'addCourseListeners']),
    getDocumentName
  }
})
export default class LegalRequestItemFull extends mixins(AdmissionsMixin) {
  readonly course!: ISingleRequestCourse;
  readonly students!: Array<IRequestStudent>;
  readonly activeProfile!: IBusinessProfileItem;

  readonly removeListener!: (payload: { flow_id: number, business_order_id: number, profile_id: number }) => Promise<void>;
  readonly addCourseListeners!: (payload: { flow_id: number, business_order_id: number, students: number[] }) => Promise<void>;
  readonly getDocumentName!: (type: string) => string;

  otherDocTypes: Array<IOtherDocType> = [];

  activeFilters = [] as Array <{ field: string, filter: string }>;
  allStudents = [] as Array<IStudent>;
  filteredStudents = [] as Array<IRequestStudent>;
  studentsToAdd = [] as Array<IStudent>;
  selectedStudents = [] as number[];

  get companyInfo () {
    return { type_company: this.activeProfile.type, company_id: this.activeProfile.id };
  };

  get businessOrderId (): number {
    return parseInt(this.$route.params.id);
  };

  async beforeMount () {
    const result = await this.$api.documents.getOtherDocTypes();
    if (!result.success) {
      this.$toasts.add(result.message, 'red');
    } else {
      this.otherDocTypes = result.data;
    }

    this.resetFilters();
    const employeesResponse = await this.$api.businessProfile.getEmployees(this.activeProfile.id);
    if (!employeesResponse.success) {
      this.$toasts.add(employeesResponse.message || 'Ошибка при получении списка пользователей', 'red');
    } else {
      this.allStudents = employeesResponse.data
        .map(employee => {
          let employee_name = '';
          if (employee.lastname) {
            employee_name += `${employee.lastname} `;
          }
          if (employee.name) {
            employee_name += `${employee.name} `;
          }
          if (employee.middle_name) {
            employee_name += employee.middle_name;
          }

          return {
            id: employee.profile_id,
            name: employee_name || 'Не указано',
            status: employee.status,
            selected: false
          };
        });
      this.studentsToAdd = this.allStudents.filter(employee => {
        return !this.students.find((el: IRequestStudent) => el.profile_id === employee.id);
      });
    }
  };

  receivedDocuments (docs: Array<IReceiveDocuments>) {
    return docs.map(el => el.title).join(', ');
  };

  async removeStudents (student: IRequestStudent) {
    await this.removeListener({
      flow_id: this.course.flow_id,
      business_order_id: this.businessOrderId,
      profile_id: student.profile_id
    });
    this.filteredStudents = this.filteredStudents.filter((el: IRequestStudent) => el.profile_id !== student.profile_id);
    const studentToAdd = this.allStudents.find(el => el.id === student.profile_id);
    if (studentToAdd) {
      this.studentsToAdd.push(studentToAdd);
    }
  };

  filtersForTable (field: string) {
    return [...new Set(
      this.students?.map((el: IRequestStudent) => {
        return el[field as keyof IRequestStudent];
      }))
    ];
  };

  sortTable (field: StudentRequestFields, direction: string) {
    this.filteredStudents = this.filteredStudents.sort((el1: IRequestStudent, el2: IRequestStudent) => {
      if (el1[field] === el2[field]) {
        return 0;
      }

      if (field === 'full_name') {
        if (direction === 'asc') {
          return el1.full_name[0].toLowerCase() > el2.full_name[0].toLowerCase() ? 1 : -1;
        } else {
          return el2.full_name[0].toLowerCase() > el1.full_name[0].toLowerCase() ? 1 : -1;
        }
      }

      // TODO фильтр для статусов
      // if (field === 'status_documents') {
      //   if (direction === 'asc') {
      //     return el1.full_name[0].toLowerCase() > el2.full_name[0].toLowerCase() ? 1 : -1;
      //   } else {
      //     return el2.full_name[0].toLowerCase() > el1.full_name[0].toLowerCase() ? 1 : -1;
      //   }
      // }

      if (direction === 'asc') {
        return el1[field] > el2[field] ? 1 : -1;
      } else {
        return el2[field] > el1[field] ? 1 : -1;
      }
    });
  };

  filterTable (field: StudentRequestFields, filter: { active: boolean, filter: string }) {
    if (filter.active) {
      this.activeFilters.push({
        field,
        filter: filter.filter
      });
    } else {
      this.activeFilters.splice(this.activeFilters.findIndex(el => {
        return el.field === field && el.filter === filter.filter;
      }), 1);
    }
    if (this.activeFilters.length) {
      this.filteredStudents = [];
      this.activeFilters.forEach(
        item => {
          this.filteredStudents.push(...this.students.filter((el: IRequestStudent) => {
            if (field === 'status_documents') {
              // TODO фильтр статусов
              return false;
            } else {
              return el[field] === item.filter;
            }
          }));
        });
    } else {
      this.resetFilters();
    }
    this.filteredStudents = [...new Set(this.filteredStudents)];
  };

  searchInCoursesTable (field: StudentRequestFields, query: string) {
    this.filteredStudents = this.students.filter((el: IRequestStudent) => {
      return el[field].toString().includes(query);
    });
  };

  async selectStudent (_: number, studentsId: number[]) {
    try {
      await this.addCourseListeners({
        flow_id: parseInt(this.$route.params.course),
        business_order_id: this.businessOrderId,
        students: studentsId
      });
      this.studentsToAdd = this.studentsToAdd.filter(employee => {
        return !this.students.find((el: IRequestStudent) => el.profile_id === employee.id);
      });
      this.$root.$emit('add-students-modal:modal:close');
    } catch (e) {
      console.error(e);
    }
  };

  resetFilters () {
    this.activeFilters = [];
    this.filteredStudents = [];
    this.students.forEach((el: IRequestStudent) => this.filteredStudents.push(el));
  };
};
</script>

<style lang="scss" module>
@import './LegalRequestItem.module';
</style>

<style lang="scss" scoped>
.col-name {
  width: 232px;
}

.col-dates {
  width: 152px;
}

.col-status {
  width: 172px;
}

.col-action {
  width: 86px;
}

.docs-message {
  width: 445px;
  padding: 16px;
}

@media screen and (min-width: $xl-breakpoint) {
  .col-name {
    width: 305px;
  }

  .col-dates {
    width: 402px;
  }

  .col-status {
    width: 228px;
  }
}
</style>
