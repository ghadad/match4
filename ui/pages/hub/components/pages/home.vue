<template>
  <section class="hero is-info is-fullheight entry-page">
    <div class="hero-head">
      <div class="container">
        <h1 class="title is-1">Datahub</h1>
        <h2
          class="subtitle"
        >Build your DATAHUB faster and with more agile data integration, while allowing real-time concurrent interactive access to data. For example, shared services and analytics reports can access the data hub at the same time, even while additional data is integrated into the data hub.</h2>
        <router-link :to="{name:'projects'}" class="button is-rounded">Explores hub projects</router-link>
        <router-link :to="{name:'datasetsIndex'}" class="button is-rounded">Smart dynamic dataset</router-link>
      </div>
    </div>
  </section>
</template>
<style>
.button {
  margin-right: 10px;
}
.hero.entry-page {
  padding-left: 30px;
}
</style>