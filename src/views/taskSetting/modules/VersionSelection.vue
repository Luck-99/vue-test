<template>
  <div class="task-setting-version-selection">
    <div class="top-title borderBottom">{{ title }}</div>
    <div v-for="i in allPackages" :key="i._id" class="packages-list">
      <span class="package-name">{{ filterLabel(i.name) }}</span>
      <el-select
        v-model="i.version"
        filterable
        @visible-change="(e) => getPackageInfo(e, i.name)"
        :loading="selectLoading"
      >
        <el-option
          v-for="item in options[i.name]"
          :key="item"
          :label="item"
          :value="item"
        >
        </el-option>
      </el-select>
    </div>
  </div>
</template>

<script>
import { getAllPackages, getPackageInfo } from "@/services/verdaccio"
import { getEnvDeps } from "@/services/file"
export default {
  name: "VersionSelection",
  data() {
    return {
      title: "依赖包版本选择",
      allPackages: [],
      selectLoading: false,
      options: {},
    }
  },
  methods: {
    async getAllPackages() {
      const packageRes = await getAllPackages()
      const tempEnvRes = await getEnvDeps({
        key: this.$store.getters.envInfo.key,
      })
      if (packageRes.code > 0) {
        this.allPackages = packageRes.data.filter((item) =>
          item.name.includes("@zglib/product-system")
        )
      }
      if (tempEnvRes.code > 0) {
        for (const [key, value] of Object.entries(tempEnvRes.data)) {
          this.allPackages = this.allPackages.map((item) => {
            const tempObj = { ...item }
            if (tempObj.name === key) {
              tempObj.version = value
            }
            return tempObj
          })
        }
      }
    },
    async getPackageInfo(show, name) {
      if (show && !this.options[name]) {
        this.selectLoading = true
        const res = await getPackageInfo({ name })
        if (res.code > 0) {
          const tempVersion = res.data.versions
          const versions = Object.keys(tempVersion)
          this.options[name] = versions
        }
        this.selectLoading = false
      }
    },
    filterLabel: (label) => {
      return label?.replace("@zglib/", "")
    },
  },
  mounted() {
    this.getAllPackages()
  },
  watch: {
    allPackages: {
      deep: true,
      handler(val) {
        const tempPackages = {}
        val.forEach((item) => {
          tempPackages[item.name] = item.version
        })
        this.$store.commit("setPackageList", tempPackages)
      },
    },
  },
}
</script>

<style lang="less" scoped>
.task-setting-version-selection {
  background-color: #fff;
  .top-title {
    text-align: left;
    padding: 14px;
  }
  .packages-list {
    padding: 7px 0;
    text-align: start;
    .package-name {
      width: 400px;
      display: inline-block;
      text-align: end;
      margin-right: 14px;
    }
  }
}
</style>
