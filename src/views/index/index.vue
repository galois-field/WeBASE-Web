<template>
  <div style="height: 100%">
    <newMain v-if="deployType == 0"></newMain>
    <mains v-if="deployType == 1"></mains>
  </div>
</template>

<script>
import newMain from "./newMain";
import mains from "./main";
import { simulateLogin, getDeployType, login } from "@/util/api";
const sha256 = require("js-sha256").sha256;
export default {
  name: "index",
  components: {
    newMain,
    mains,
  },
  data: function () {
    return {
      deployType: null,
    };
  },
  created: function () {
    if (localStorage.getItem("token")) {
      if (localStorage.getItem("deployType")) {
        this.deployType = localStorage.getItem("deployType");
      } else {
        this.deployType = 0;
      }
      return;
    }
    let urlStr = location.href.split("?")[1];
    const urlSearchParams = new URLSearchParams(urlStr);
    const result = Object.fromEntries(urlSearchParams.entries());
    console.log(result);
    if (!result.token) {
      this.$message({
        message: "未找到token",
        type: "error",
        duration: 2000,
      });
      return;
    }
    const params = {
      access_token: result.token,
    };
    simulateLogin(params).then((res) => {
      if (res.data.code == 1) {
        if (!res.data.data.user.loginName) {
          this.$message({
            message: "Not found loginName",
            type: "error",
            duration: 2000,
          });
        }
        this.userLogin(res.data.data.user.loginName, result.token);
      } else {
        this.$message({
          message: "System error",
          type: "error",
          duration: 2000,
        });
      }
    });
    localStorage.removeItem("selectData");
  },
  methods: {
    userLogin(account, access_token) {
      const reqData = {
        account: account,
        accountPwd: sha256(account),
      };
      login(reqData, access_token).then((res) => {
        if (res.data.code === 0) {
          localStorage.setItem("groupName", "");
          localStorage.setItem("groupId", "");
          localStorage.setItem("folderList", "");
          localStorage.setItem("user", res.data.data.account);
          localStorage.setItem("root", res.data.data.roleName);
          localStorage.setItem("token", res.data.data.token);
          sessionStorage.setItem("accountStatus", res.data.data.accountStatus);
          sessionStorage.setItem("reload", 1);
          localStorage.setItem("config", 0);
          localStorage.setItem("nodeVersionChange", "");
          localStorage.setItem("selectData", "");
          localStorage.setItem("solcName", "");
          localStorage.setItem("versionId", null);
          this.getConfigType();
        }
      });
    },
    getConfigType: function () {
      getDeployType()
        .then((res) => {
          if (res.data.code == 0) {
            localStorage.setItem("deployType", res.data.data);
            // router.push("/main");
            this.$router.go(0)
          } else {
            this.$message({
              message: this.$chooseLang(res.data.code),
              type: "error",
              duration: 2000,
            });
          }
        })
        .catch((err) => {
          this.$message({
            message: this.$t("text.systemError"),
            type: "error",
            duration: 2000,
          });
        });
    },
  },
};
</script>

<style>
</style>