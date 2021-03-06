<template>
  <div>
    <div class="page-title">
      <h3>{{ "Planing_title" | localize }}</h3>
      <h4>{{ info.bill | currencyFilter("RUB") }}</h4>
    </div>

    <Loader v-if="loading" />
    <p v-else-if="!categories.length" class="center">
      {{ "Planing_no_categories" | localize }}
      <router-link to="/categories">
        {{ "Planing_add_category" | localize }}</router-link
      >
    </p>

    <section class="progress-Bar" v-else>
      <div v-for="cat of categories" :key="cat.id" v-tooltip="cat.tooltip">
        <p>
          <strong>{{ cat.title }}</strong>
          {{ cat.spend | currencyFilter }} из {{ cat.limit | currencyFilter }}
        </p>
        <div class="progress">
          <div
            class="determinate green"
            :class="[cat.progressColor]"
            :style="{ width: cat.progressPercent + '%' }"
          ></div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import currencyFilter from "@/filters/currency.filter";
import localizeFilter from "@/filters/localize.filter";

export default {
  data: () => ({ loading: true, categories: [] }),
  async mounted() {
    const records = await this.$store.dispatch("fetchRecords");
    const categories = await this.$store.dispatch("fetchCategories");

    this.categories = categories.map(cat => {
      const spend = records
        .filter(f => f.categoryId == cat.id)
        .filter(f => f.type === "outcome")
        .reduce((total, record) => {
          return (total += +record.amount);
        }, 0);

      const percent = (100 * spend) / cat.limit;
      const progressPercent = percent > 100 ? 100 : percent;
      const progressColor =
        percent < 60 ? "green" : percent < 100 ? "yellow" : "red";

      const tooltipValue = cat.limit - spend;
      const tooltip = `${
        tooltipValue < 0
          ? localizeFilter("tooltip_excess_money")
          : localizeFilter("tooltip_balance")
      } ${currencyFilter(Math.abs(tooltipValue))}`;

      return {
        ...cat,
        progressPercent,
        progressColor,
        spend,
        tooltip
      };
    });

    this.loading = false;
  },

  computed: {
    ...mapGetters(["info"])
  }
};
</script>
<style scoped>
.progress-Bar {
  cursor: pointer;
}
</style>
