<template>
  <div>
    <el-row style="margin-top: 10px">
      <el-col>
        <el-card>
          <div slot="header" class="header">
            <span @click="goToProject" style="cursor: pointer"> Subscription Converter </span>
            <div class="backend-container">
              <span
                :style="{ color: ping === -1 ? 'red' : '#333' }"
                style="cursor: pointer"
                @click="goToBackendProject"
              >
                {{ backendVersion }}
              </span>
              <span
                v-if="ping && ping !== -1"
                :class="['ping', ping < 1000 ? 'good' : ping < 3000 ? 'normal' : 'bad']"
                @click="getBackendVersion"
              >
                {{ ping }}ms
              </span>
            </div>
          </div>
          <el-container>
            <el-form :model="form" label-width="80px" label-position="left" style="width: 100%">
              <el-form-item label="模式设置:">
                <el-radio v-model="advanced" label="1">基础模式</el-radio>
                <el-radio v-model="advanced" label="2">进阶模式</el-radio>
              </el-form-item>
              <el-form-item label="订阅链接:">
                <el-input
                  v-model="form.sourceSubUrl"
                  type="textarea"
                  rows="3"
                  placeholder="支持订阅或ss/ssr/vmess链接，多个链接每行一个或用 | 分隔"
                  @blur="saveSubUrl"
                />
              </el-form-item>
              <el-form-item label="客户端:">
                <el-select v-model="form.clientType" style="width: 100%">
                  <el-option v-for="(v, k) in options.clientTypes" :key="k" :label="k" :value="v"></el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="后端地址:">
                <el-select
                  v-model="form.customBackend"
                  allow-create
                  filterable
                  placeholder="请选择"
                  style="width: 100%"
                  @change="backendChange"
                >
                  <el-option
                    v-for="item in options.backendOptions"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  />
                </el-select>
                <!-- <el-autocomplete
                  style="width: 100%"
                  v-model="form.customBackend"
                  :fetch-suggestions="backendSearch"
                  placeholder="动动小手，（建议）自行搭建后端服务。例：http://127.0.0.1:25500/sub?"
                >
                  <el-button slot="append" @click="gotoGayhub" icon="el-icon-link">前往项目仓库</el-button>
                </el-autocomplete> -->
              </el-form-item>
              <el-form-item label="远程配置:">
                <el-select v-model="form.remoteConfig" allow-create filterable placeholder="请选择" style="width: 100%">
                  <el-option-group v-for="group in options.remoteConfig" :key="group.label" :label="group.label">
                    <el-option
                      v-for="item in group.options"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-option-group>
                </el-select>
              </el-form-item>
              <div v-if="advanced === '2'">
                <el-form-item label="Include:">
                  <el-input v-model="form.includeRemarks" placeholder="节点名包含的关键字，支持正则" />
                </el-form-item>
                <el-form-item label="Exclude:">
                  <el-input v-model="form.excludeRemarks" placeholder="节点名不包含的关键字，支持正则" />
                </el-form-item>
                <el-form-item label="FileName:">
                  <el-input v-model="form.filename" placeholder="返回的订阅文件名" />
                </el-form-item>
                <el-form-item label-width="0px">
                  <el-row type="flex">
                    <el-col>
                      <el-checkbox v-model="form.nodeList" label="输出为 Node List" border></el-checkbox>
                    </el-col>
                    <el-popover placement="bottom" v-model="form.extraset">
                      <el-row>
                        <el-checkbox v-model="form.emoji" label="Emoji"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.scv" label="跳过证书验证"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.udp" @change="needUdp = true" label="启用 UDP"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.appendType" label="节点类型"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.sort" label="排序节点"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.fdn" label="过滤非法节点"></el-checkbox>
                      </el-row>
                      <el-button slot="reference">更多选项</el-button>
                    </el-popover>
                    <el-popover placement="bottom" style="margin-left: 10px">
                      <el-row>
                        <el-checkbox v-model="form.tpl.surge.doh" label="Surge.DoH"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.tpl.clash.doh" label="Clash.DoH"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.insert" label="网易云"></el-checkbox>
                      </el-row>
                      <el-button slot="reference">定制功能</el-button>
                    </el-popover>
                  </el-row>
                </el-form-item>
              </div>

              <el-checkbox v-model="form.proxy" label="使用代理拉取订阅（获取不到节点时请尝试勾选）" border />

              <div style="margin-top: 50px"></div>

              <el-divider content-position="center">
                <i class="el-icon-magic-stick"></i>
              </el-divider>

              <el-form-item label="定制订阅:">
                <el-input class="copy-content" disabled v-model="customSubUrl">
                  <el-button
                    slot="append"
                    v-clipboard:copy="customSubUrl"
                    v-clipboard:success="onCopy"
                    ref="copy-btn"
                    icon="el-icon-document-copy"
                    >复制</el-button
                  >
                </el-input>
              </el-form-item>
              <el-form-item label="订阅短链:">
                <el-input class="copy-content" disabled v-model="customShortSubUrl">
                  <el-button
                    slot="append"
                    v-clipboard:copy="customShortSubUrl"
                    v-clipboard:success="onCopy"
                    ref="copy-btn"
                    icon="el-icon-document-copy"
                    >复制</el-button
                  >
                </el-input>
              </el-form-item>
              <el-form-item label-width="0px" style="margin-top: 40px; text-align: center">
                <el-button
                  style="width: 120px"
                  type="danger"
                  @click="makeUrl"
                  :disabled="form.sourceSubUrl.length === 0"
                  >生成订阅链接</el-button
                >
                <!-- <el-button
                  style="width: 120px"
                  type="danger"
                  @click="makeShortUrl"
                  :loading="loading"
                  :disabled="customSubUrl.length === 0"
                  >生成短链接</el-button
                > -->
              </el-form-item>

              <el-form-item label-width="0px" style="text-align: center">
                <el-button
                  style="width: 120px"
                  type="primary"
                  @click="clashInstall"
                  icon="el-icon-connection"
                  :disabled="customSubUrl.length === 0"
                  >一键导入Clash</el-button
                >
              </el-form-item>
            </el-form>
          </el-container>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
const project = process.env.VUE_APP_PROJECT;
const backendProject = process.env.VUE_APP_BACKEND_PROJECT;
const gayhubRelease = process.env.VUE_APP_BACKEND_RELEASE;
const defaultBackend = process.env.VUE_APP_SUBCONVERTER_DEFAULT_BACKEND + "/subconverter/sub?";
const shortUrlBackend = process.env.VUE_APP_SUBCONVERTER_DEFAULT_BACKEND + "/short/create";
const defaultConfig = "https://raw.githubusercontent.com/7revor/proxy-rules/main/subconverter.lite.ini";
export default {
  data() {
    return {
      backendVersion: "",
      // 是否为 PC 端
      isPC: true,
      advanced: "1",
      options: {
        clientTypes: {
          Clash: "clash",
          Surge3: "surge&ver=3",
          Surge4: "surge&ver=4",
          Quantumult: "quan",
          QuantumultX: "quanx",
          Surfboard: "surfboard",
          Loon: "loon",
          SSAndroid: "sssub",
          V2Ray: "v2ray",
          ss: "ss",
          ssr: "ssr",
          ssd: "ssd",
          ClashR: "clashr",
          Surge2: "surge&ver=2",
        },
        backendOptions: [{ label: "https://api.k1kyo.com", value: defaultBackend }],
        remoteConfig: [
          {
            label: "CUSTOMIZED",
            options: [
              {
                label: "7revor（精简）",
                value: "https://raw.githubusercontent.com/7revor/proxy-rules/main/subconverter.lite.ini",
              },
              {
                label: "7revor（自用）",
                value: "https://raw.githubusercontent.com/7revor/proxy-rules/main/subconverter.min.ini",
              },
            ],
          },
          {
            label: "ACL4SSR",
            options: [
              {
                label: "默认版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online.ini",
              },
              {
                label: "去广告",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_AdblockPlus.ini",
              },
              {
                label: "多国分组",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_MultiCountry.ini",
              },
              {
                label: "无自动测速",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoAuto.ini",
              },
              {
                label: "无广告拦截",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoReject.ini",
              },
              {
                label: "精简版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini.ini",
              },
              {
                label: "精简版去广告",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_AdblockPlus.ini",
              },
              {
                label: "精简版无测速",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_NoAuto.ini",
              },
              {
                label: "精简版故障转移",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_Fallback.ini",
              },
              {
                label: "精简版(测速|故障转移|负载均衡)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_MultiMode.ini",
              },
              {
                label: "精简版(港美日)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_MultiCountry.ini",
              },
              {
                label: "全分组",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full.ini",
              },
              {
                label: "全分组多模式",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_MultiMode.ini",
              },
              {
                label: "全分组无测速",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_NoAuto.ini",
              },
              {
                label: "全分组去广告",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_AdblockPlus.ini",
              },
            ],
          },
        ],
      },
      form: {
        sourceSubUrl: "",
        clientType: "",
        customBackend: defaultBackend,
        remoteConfig: defaultConfig,
        excludeRemarks: "",
        includeRemarks: "",
        filename: "",
        emoji: false,
        proxy: false,
        extraset: false,
        sort: false,
        udp: false,
        nodeList: false,
        tfo: false,
        scv: false,
        fdn: false,
        appendType: false,
        insert: false, // 是否插入默认订阅的节点，对应配置项 insert_url
        new_name: true, // 是否使用 Clash 新字段

        // tpl 定制功能
        tpl: {
          surge: {
            doh: false, // dns 查询是否使用 DoH
          },
          clash: {
            doh: false,
          },
        },
      },

      loading: false,
      customSubUrl: "",
      customShortSubUrl: "",

      loadConfig: "",
      uploadConfig: "",
      uploadPassword: "",

      needUdp: false, // 是否需要添加 udp 参数

      ping: "", // 服务器延迟
    };
  },
  created() {
    document.title = "Subscription Converter";
    this.isPC = this.$getOS().isPc;
    // 获取 url cache
    if (process.env.VUE_APP_USE_STORAGE === "true") {
      this.form.sourceSubUrl = this.getLocalStorageItem("sourceSubUrl");
    }
  },
  mounted() {
    this.form.clientType = "clash";
    this.getBackendVersion();
  },
  methods: {
    onCopy() {
      this.$message.success("Copied!");
    },
    goToProject() {
      window.open(project);
    },
    goToBackendProject() {
      window.open(backendProject);
    },
    gotoGayhub() {
      window.open(gayhubRelease);
    },
    clashInstall() {
      if (this.customSubUrl === "") {
        this.$message.error("请先填写必填项，生成订阅链接");
        return false;
      }

      const url = "clash://install-config?url=";
      window.open(url + encodeURIComponent(this.customShortSubUrl !== "" ? this.customShortSubUrl : this.customSubUrl));
    },
    surgeInstall() {
      if (this.customSubUrl === "") {
        this.$message.error("请先填写必填项，生成订阅链接");
        return false;
      }

      const url = "surge://install-config?url=";
      window.open(url + this.customSubUrl);
    },
    makeUrl() {
      if (this.form.sourceSubUrl === "" || this.form.clientType === "") {
        this.$message.error("订阅链接与客户端为必填项");
        return false;
      }

      let backend = this.form.customBackend === "" ? defaultBackend : this.form.customBackend;

      let sourceSub = this.form.sourceSubUrl;
      sourceSub = sourceSub.replace(/(\n|\r|\n\r)/g, "|");
      // 使用订阅代理
      if (this.form.proxy) {
        sourceSub = sourceSub
          .split("|")
          .map((url) => {
            if (url.startsWith("http")) {
              return `https://pfetch.k1kyo.com/?url=${encodeURIComponent(url)}`;
            } else {
              return url;
            }
          })
          .join("|");
      }

      this.customSubUrl =
        backend +
        "target=" +
        this.form.clientType +
        "&url=" +
        encodeURIComponent(sourceSub) +
        "&insert=" +
        this.form.insert;

      if (this.form.excludeRemarks !== "") {
        this.customSubUrl += "&exclude=" + encodeURIComponent(this.form.excludeRemarks);
      }
      if (this.form.includeRemarks !== "") {
        this.customSubUrl += "&include=" + encodeURIComponent(this.form.includeRemarks);
      }
      if (this.form.filename !== "") {
        this.customSubUrl += "&filename=" + encodeURIComponent(this.form.filename);
      }
      if (this.form.appendType) {
        this.customSubUrl += "&append_type=" + this.form.appendType.toString();
      }
      if (this.advanced === "2") {
        this.customSubUrl +=
          "&emoji=" +
          this.form.emoji.toString() +
          "&list=" +
          this.form.nodeList.toString() +
          "&tfo=" +
          this.form.tfo.toString() +
          "&scv=" +
          this.form.scv.toString() +
          "&fdn=" +
          this.form.fdn.toString() +
          "&sort=" +
          this.form.sort.toString();
      }
      if (this.needUdp) {
        this.customSubUrl += "&udp=" + this.form.udp.toString();
      }

      if (this.form.tpl.surge.doh === true) {
        this.customSubUrl += "&surge.doh=true";
      }

      if (this.form.clientType === "clash") {
        if (this.form.tpl.clash.doh === true) {
          this.customSubUrl += "&clash.doh=true";
        }
        this.customSubUrl += "&new_name=" + this.form.new_name.toString();
      }
      if (this.form.remoteConfig !== "") {
        this.customSubUrl += "&config=" + encodeURIComponent(this.form.remoteConfig);
      }

      this.$copyText(this.customSubUrl);
      this.$message.success("定制订阅已复制到剪贴板");
    },
    makeShortUrl() {
      if (this.customSubUrl === "") {
        this.$message.warning("请先生成订阅链接，再获取对应短链接");
        return false;
      }
      this.loading = true;
      this.$axios
        .post(
          shortUrlBackend,
          { target: this.customSubUrl, reuse: true },
          {
            header: {
              "Content-Type": "application/json",
            },
          }
        )
        .then((res) => {
          this.customShortSubUrl = res.data.link;
          this.$copyText(res.data.link);
          this.$message.success("短链接已复制到剪贴板");
        })
        .catch((err) => {
          const msg = err.response ? err.response.data.error : err.message;
          this.$message.error("短链接获取失败：" + msg);
        })
        .finally(() => {
          this.loading = false;
        });
    },
    backendChange() {
      this.getBackendVersion();
    },
    getBackendVersion() {
      this.backendVersion = "";
      this.ping = "";
      const start = new Date().getTime();
      const backend = this.form.customBackend;
      this.$axios
        .get(backend.substring(0, backend.length - 5) + "/version")
        .then((res) => {
          const end = new Date().getTime();
          this.backendVersion = res.data.replace(/backend\n$/gm, "");
          this.backendVersion = this.backendVersion.replace("subconverter", "");
          this.ping = end - start;
        })
        .catch((err) => {
          this.backendVersion = err.message;
          this.ping = -1;
        });
    },
    saveSubUrl() {
      if (this.form.sourceSubUrl !== "") {
        this.setLocalStorageItem("sourceSubUrl", this.form.sourceSubUrl);
      }
    },
    getLocalStorageItem(itemKey) {
      const now = +new Date();
      let ls = localStorage.getItem(itemKey);

      let itemValue = "";
      if (ls !== null) {
        let data = JSON.parse(ls);
        if (data.expire > now) {
          itemValue = data.value;
        } else {
          localStorage.removeItem(itemKey);
        }
      }

      return itemValue;
    },
    setLocalStorageItem(itemKey, itemValue) {
      const ttl = process.env.VUE_APP_CACHE_TTL;
      const now = +new Date();

      let data = {
        setTime: now,
        ttl: parseInt(ttl),
        expire: now + ttl * 1000,
        value: itemValue,
      };
      localStorage.setItem(itemKey, JSON.stringify(data));
    },
  },
};
</script>
<style lang="css">
.ping {
  background-color: green;
  border-radius: 3px;
  padding: 4px;
  color: #fff;
  margin-left: 4px;
  cursor: pointer;
  line-height: 12px;
  font-size: 12px;
}
.ping.normal {
  background-color: orange;
}
.ping.bad {
  background-color: red;
}
.backend-container {
  font-size: 14px;
  display: flex;
  align-items: center;
}
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 20px;
}
</style>
