<template>
  <div class="main">
    <Banner />
    <div class="container"></div>
    <Header />
    <div v-if="!loading" class="content">
      <div class="maybeGonnaMakeIt">
        <div>If you start today by staking only 1 Ethereal...</div>
        <div :class="{ notGonnaMakeIt: !gonnaMakeIt, gonnaMakeIt }">
          YOU'RE {{ gonnaMakeIt ? "GONNA MAKE IT!" : "not gonna make it." }}
        </div>
        <div>
          <span v-if="!gonnaMakeIt" class="minimumStaking"
            >Stake at least {{ minimum }} ghosts to get an origin</span
          >
        </div>
        <div><a href="/amigonnamakeit">But are you gonna make it?</a></div>
      </div>
      <div class="stats">
        <Counts :stats="stats" />
      </div>
      <div class="table">
        <div class="title">Current staking counts</div>
        <Table :rows="rows" :headers="headers" :keys="keys" />
      </div>
    </div>
    <Spinner v-else />
    <div class="footer">
      * These are approximations and not exact numbers
      <br />
      Built by <a href="https://twitter.com/philosowrapter">@philoswrapter</a>
      <br />
      No need but if you feel like it:
      0x9194851F2A28e209216bEB93b736Ff3cF8579948
    </div>
  </div>
</template>



<script>
import Header from "../components/header.vue";
import Counts from "../components/counts.vue";
import Spinner from "../components/spinner.vue";
import Table from "../components/table.vue";
import axios from "axios";
export default {
  name: "IndexPage",
  data: () => ({
    loading: true,
    stats: {},
    alreadyMinted: 0,
    gonnaMakeIt: false,
    rows: [],
    headers: ["# of Ghosts", "# of Stakers", "Days Until Mint"],
    keys: ["ghosts", "holders", "daysTilMint"],
  }),
  components: {
    Counts,
    Spinner,
    Table,
    Header,
  },
  methods: {
    async fetchStats() {
      try {
        const res = await axios.get("https://staking-api.guanaco.dev/stats");
        this.stats = res?.data?.data;

        this.rows = this.stats.totals.sort((a, b) => a.ghosts - b.ghosts);

        this.loading = false;
      } catch (err) {
        console.log(err);
      }
    },
    async getAllAOrigins() {
      const res = await axios.get(
        "https://staking-api.guanaco.dev/totalOrigins"
      );
      this.alreadyMinted = res?.data?.data;
    },
  },
  async mounted() {
    await Promise.all([this.fetchStats(), this.getAllAOrigins()]);
    this.gonnaMakeIt =
      this.stats?.mintsBeforeDay120 + this.alreadyMinted < 4116;
  },
};
</script>

<style lang="scss">
.main {
  // max-width: 60vw;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: auto;

  .header {
    font-weight: bold;
    font-size: 36px;
    // white-space: nowrap;
  }
  .maybeGonnaMakeIt {
    display: flex;
    font-weight: bold;
    font-size: 36px;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  .gonnaMakeIt {
    margin-top: 15px;
    background-color: $green;
    padding: 3px 10px;
    border-radius: 5px;
  }
  .notGonnaMakeIt {
    margin-top: 15px;
    background-color: $orange;
    padding: 3px 10px;
    border-radius: 5px;
  }
  .stats {
    margin-top: 30px;
  }

  .table {
    margin-top: 30px;
    .title {
      font-weight: bold;
      margin-bottom: 10px;
      font-size: 18px;
    }
  }
  .footer {
    margin-top: 10px;
  }
}

@media only screen and (max-width: 500px) {
  .main {
    .header {
      font-size: 24px;
    }
    .maybeGonnaMakeIt {
      font-size: 18px;
    }
  }
}
</style>