/*
*
* Сделано в студии AppFox
*
*/
<template>
  <div
    :class="$style.RequestItem"
  >
    <div
      class="d-flex"
      :class="$style.RequestTop"
    >
      <div
        class="mr-16"
        :class="$style.RequestMainInfo"
      >
        <div
          class="d-flex justify-content-between mb-8"
          :class="$style.RequestMainInfoTop"
        >
          <div class="green-color small-text">
            ID {{ data.id }}
          </div>

          <div class="gray-color small-text">
            {{ data.start_edu }} - {{ data.end_edu }}
          </div>
        </div>

        <nuxt-link
          v-if="!$route.params.course"
          :to="requestLink"
          class="subtitle base-text-color text-decoration-none"
        >
          {{ data.title }}
        </nuxt-link>

        <div class="mt-8 d-flex align-items-center flex-wrap">
          <div
            class="green-color subhead mr-8"
            :class="$style.RequestItemPrice"
          >
            {{ data.full_price_for_this_course }} руб. / {{ data.count_students }} чел.
          </div>

          <Tag
            color="white"
            size="xs"
          >
            {{ data.study_form }}
          </Tag>
        </div>
      </div>

      <div
        :class="$style.RequestConrols"
        class="d-flex"
      >
        <nuxt-link
          v-if="!full"
          :to="requestLink"
          class="mr-8"
        >
          <button class="btn-icon btn-icon-white btn-icon-color-green">
            <Icon name="pencil" />
          </button>
        </nuxt-link>

        <button
          class="btn-icon btn-icon-white mb-8"
          @click="removeCourseHandler"
        >
          <Icon name="trash" />
        </button>
      </div>
    </div>

    <div
      :class="[$style.RequestSubInfoHeader, $route.params.course ? $style.RequestSubInfoHeaderList : '']"
      class="d-flex"
    >
      <div :class="$style.RequestSubInfoHeaderItem">
        <div class="gray-label">
          Направление:
        </div>

        <div class="small-text">
          {{ data.direction }}
        </div>
      </div>

      <div :class="$style.RequestSubInfoHeaderItem">
        <div class="gray-label">
          Специальность:
        </div>

        <div class="small-text">
          {{ data.speciality }}
        </div>
      </div>

      <nuxt-link
        v-if="fullIcon && course"
        :to="`/catalog/${course.slug}`"
        class="btn-icon btn-icon-green pull-right"
      >
        <Icon name="link" />
      </nuxt-link>
    </div>

    <slot></slot>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator';
import { mapActions, mapState } from 'vuex';
import { ISingleRequestCourse } from '~/types/interfaces/Request.interface';

@Component({
  computed: mapState('request', ['course']),
  methods: mapActions('request', ['removeCourse'])
})
export default class LegalRequestItem extends Vue {
  @Prop({ type: Object, default: () => {} })
  readonly data!: ISingleRequestCourse;

  @Prop({ type: Boolean, default: false })
  readonly full!: boolean;

  @Prop({ type: Boolean, default: false })
  readonly fullIcon!: boolean;

  readonly course!: ISingleRequestCourse | null;

  readonly removeCourse!: (payload: { flow_id: number, order_id: number }) => Promise<void>;

  get requestLink (): string {
    return `/profile/orders/${this.$route.params.id}/${this.data.flow_id}`;
  };

  async removeCourseHandler () {
    await this.removeCourse({ order_id: parseInt(this.$route.params.id), flow_id: this.data.flow_id });
    if (this.full) {
      this.$router.push(`/profile/orders/${this.$route.params.id}`);
    }
  };
};
</script>

<style lang="scss" module>
@import './LegalRequestItem.module';
</style>
