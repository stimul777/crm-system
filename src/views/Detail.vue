<template>
  <div>
    <Loader v-if="loading" />
    <div v-else-if="record">
      <div class="breadcrumb-wrap">
        <router-link to="/history" class="breadcrumb">{{
          "Detail" | localize
        }}</router-link>
        <a class="breadcrumb" @click.prevent>
          {{ record.type === "income" ? "Доход" : "Расход" }}
        </a>
      </div>

      <div class="row">
        <div class="col s12 m6">
          <div
            class="card"
            :class="{
              red: record.type === 'outcome',
              green: record.type === 'income'
            }"
          >
            <div class="card-content white-text">
              <p>
                {{ "Detail_description" | localize }} :
                {{ record.description }}
              </p>
              <p>
                {{ "Detail_sum" | localize }}:
                {{ record.amount | currencyFilter }}
              </p>
              <p>
                {{ "Detail_category" | localize }} : {{ record.categoryName }}
              </p>
              <small>{{ record.date | dateFilter("datetime") }}</small>
            </div>
          </div>
        </div>
      </div>
    </div>

    <p class="center" v-else>
      {{ "Detail_warning" | localize }} <strong>{{ $route.params.id }}</strong>
    </p>
  </div>
</template>

<script>
import Loader from "../components/app/Loader.vue";
export default {
  components: { Loader },
  name: "detail",

  data: () => ({
    loading: true,
    record: null
  }),

  async mounted() {
    // забрать id с адресной строки
    const id = this.$route.params.id;

    const record = await this.$store.dispatch("fetchRecordsById", id);
    const category = await this.$store.dispatch(
      "fetchCategoriyByID",
      record.categoryId
    );

    this.record = {
      ...record,
      categoryName: category.title
    };

    this.loading = false;
  }
};
</script>
