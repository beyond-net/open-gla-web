<template>
  <div v-if="tab===1">
    <el-card style="width: 680px; padding: 20px">
      <template #header>
        <div class="card-header">
          <h2>免费使用申请</h2>
        </div>
      </template>
      <div class="container">
        <el-form ref="formRef" :model="model" label-position="top" :rules="rules">
          <el-form-item label="协议" prop="protocol" required>
            <el-select v-model:model-value="model.protocol">
              <el-option label="tcp" value="tcp" />
            </el-select>
          </el-form-item>

          <el-form-item label="内网IP" prop="ip" required>
            <el-input v-model:model-value="model.ip" />
          </el-form-item>

          <el-form-item label="内网端口" prop="port" required>
            <el-input v-model.number="model.port" />
          </el-form-item>

          <el-form-item>
            <el-button type="primary" @click="handleSubmit">提交</el-button>
          </el-form-item>
        </el-form>
      </div>
      <template #footer>
        <span style="color: red"
          ><b>每日免费开放1000个测试名额，先到先得</b></span
        >
      </template>
    </el-card>
  </div>

  <div v-if="tab===2">
    <el-card style="width: 800px; padding: 20px">
      <template #header>
        <div class="card-header">
          <h2>申请成功，以下为详细信息</h2>
        </div>
      </template>
      <el-descriptions     
      direction="vertical"
      :column="2"
      border>
        <el-descriptions-item label="客户端ID">{{ result?.uuid }}</el-descriptions-item>
        <el-descriptions-item label="协议">{{ result?.protocol }}</el-descriptions-item>
        <el-descriptions-item label="公网IP/域名">{{ result?.public_ip }}</el-descriptions-item>
        <el-descriptions-item label="公网端口">{{ result?.public_port }}</el-descriptions-item>
        <el-descriptions-item label="内网IP/域名"> {{ result?.ip }}</el-descriptions-item>
        <el-descriptions-item label="内网端口">{{ result?.port }}</el-descriptions-item>
        <el-descriptions-item label="运行命令">
          <p  style="line-height: 20px;">您可以先到 <a href="https://github.com/ICKelin/zta/releases/tag/v0.0.1-mvp" target="_blank">github</a>
          下载对应的客户端文件，然后执行./zta-client_操作系统_CPU架构 -client_id={{ result?.uuid }} -server_addr=zta.beyondnetwork.net:12359
          </p>
          <br>
          mac示例:<p style="line-height: 20px;"> ./zta-client_darwin_amd64 -client_id={{ result?.uuid }} -server_addr=zta.beyondnetwork.net:12359</p>
        </el-descriptions-item>
      </el-descriptions>

      <template #footer>
        <span style="color: red"
          ><b>申请成功后配置预计在1分钟以内生效</b></span
        >
      </template>
    </el-card>
  </div>
</template>

<script lang="ts" setup>
import type {FormInstance, FormRules} from "element-plus";
import {reactive, ref} from "vue";

var tab = ref<number>(1);
interface FormModel {
  protocol: "tcp";
  ip: string;
  port: string;
}

interface ResponseModel {
  apply_hour: number;
  created_at: number;
  ip: string;
  port: number;
  protocol: string;
  public_ip: string;
  public_port: number;
  uuid: string;
}

const result = ref<ResponseModel>();

const model = reactive<FormModel>({
  protocol: "tcp",
  ip: "",
  port: "",
});

const validateIp = (rule: any, value: any, callback: any) => {
  if (value === "") {
    callback(new Error("请输入内网IP"));
  } else {
    if (model.ip !== "") {
      if (
        !/^([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])$/.test(
          model.ip
        )
      ) {
        callback(new Error("无效的IP地址"));
        return;
      }
    }
    callback();
  }
};

const rules = reactive<FormRules>({
  protocol: [
    {
      required: true,
      message: "请选择协议",
      trigger: "blur",
    },
  ],
  ip: [
    {
      required: true,
      message: "请输入内网IP",
      trigger: "blur",
    },
    {
      validator: validateIp,
      trigger: "blur",
    },
  ],
  port: [
    {
      required: true,
      message: "请输入内网端口",
      trigger: "blur",
    },
    {
      type: "number",
      min: 1,
      max: 65535,
      message: "无效的内网端口",
    },
  ],
});
const formRef = ref<FormInstance>();

const handleSubmit = () => {
  if (!formRef.value) {
    return;
  }

  formRef.value.validate(async (valid) => {
    await submit(model.protocol, model.ip, parseInt(model.port));
  });
};

const submit = async (protocol: string, ip: string, port: number) => {
  const response = await fetch("http://zta.beyondnetwork.net:12358/apply", {
    method: "POST",
    headers: {
      "Content-Type": "application/json; charset=utf-8",
    },
    body: JSON.stringify({
      protocol,
      ip,
      port,
    }),
  });

  if (!response.ok) {
    if (response.status == 409) {
      alert("当日免费额度已满，请改日再提交");
    } else {
      alert("请求出错: " + response.status);
    }
    return;
  }

  const data: ResponseModel = await response.json();
  result.value = data;
  tab.value = 2;
};
</script>

<style lang="css"></style>
