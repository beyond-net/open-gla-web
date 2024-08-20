<template>
  <el-config-provider :locale="zhCn">
    <div class="container">
      <el-form
        ref="formRef"
        :model="model"
        label-position="top"
        style="width: 400px"
        :status-icon="true"
        :rules="rules"
      >
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
  </el-config-provider>
</template>

<script lang="ts" setup>
import type { FormInstance, FormRules } from "element-plus";
import zhCn from "element-plus/es/locale/lang/zh-cn";
import { reactive, ref } from "vue";

interface FormModel {
  protocol: "tcp";
  ip: string;
  port: string;
}

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
    console.error("请求出错", response.status);
    return;
  }
  console.log(response.json());
};
</script>

<style lang="css">
.container {
  border: 1px solid gray;
  padding: 40px;
  border-radius: 20px;
}
</style>
