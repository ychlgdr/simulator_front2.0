<template>
  <div>
    <div style="background-color: rgb(255, 255, 255); height: 40px">
      <AreaTitle title="属性" style="padding-left: 15px"></AreaTitle>
    </div>
    <a-form readonly>
      <a-form-item label="名称：" :labelCol="labelCol" :wrapperCol="wrapperCol">
        <a-input
          style="font-size: 12px"
          :disabled="readOnly"
          @change="changeLabel"
          v-model="formModel.label"
        />
      </a-form-item>
      <a-form-item label="大小：" :labelCol="labelCol" :wrapperCol="wrapperCol">
        <a-slider
          :min="8"
          :max="16"
          :disabled="readOnly"
          @change="changeFontSize"
          v-model="formModel.fontSize"
        />
      </a-form-item>
    </a-form>
<div style="width: 100%;">
    <a-table :columns="columns" :data-source="data" :scroll='scroll' bordered >
      <template
        v-for="col in ['interface', 'ip', 'mask', 'gateway']"
        :slot="col"
        slot-scope="text, record"
      >
        <div :key="col">
          <a-input
            v-if="record.editable"
            style="margin: -5px 0"
            :value="text"
            @change="(e) => handleChange(e.target.value, record.key, col)"
          />
          <template v-else>
            {{ text }}
          </template>
        </div>
      </template>
      <template slot="operation" slot-scope="text, record">
        <div class="editable-row-operations">
          <span v-if="record.editable">
            <a @click="() => save(record.key)">Save</a>
            <a-popconfirm
              title="Sure to cancel?"
              @confirm="() => cancel(record.key)"
            >
              <a>Cancel</a>
            </a-popconfirm>
          </span>
          <span v-else>
            <a :disabled="editingKey !== ''" @click="() => edit(record.key)"
              >Edit</a
            >
          </span>
        </div>
      </template>
    </a-table>
</div>
    <div>
      <a-input placeholder="interface" v-model="interfaceN" />
      <a-input placeholder="ip" v-model="ipN" />
      <a-input placeholder="mask" v-model="maskN" />
      <a-input placeholder="gateway" v-model="gatewayN" />
      <a-button @click="add">Save</a-button>
    </div>
    <slot name="edge"> </slot>
  </div>
</template>
<script>
import AreaTitle from "../../AreaTitle/AreaTitle";

const columns = [
  {
    title: "interface",
    dataIndex: "interface",
    width: "15%",
    scopedSlots: { customRender: "interface" },
  },
  {
    title: "ip",
    dataIndex: "ip",
    width: "20%",
    scopedSlots: { customRender: "ip" },
    ellipsis: true,
  },
  {
    title: "mask",
    dataIndex: "mask",
    width: "20%",
    scopedSlots: { customRender: "mask" },
  },
  {
    title: "gateway",
    dataIndex: "gateway",
    width: "15%",
    scopedSlots: { customRender: "gateway" },
  },
  {
    title: "operation",
    dataIndex: "operation",
    scopedSlots: { customRender: "operation" },
  },
];

export default {
  name: "NodePanel",
  components: { AreaTitle },
  props: ["type", "readOnly", "cell", "formModel"],
  data() {
    return {
      labelCol: {
        xs: { span: 4 },
        sm: { span: 4 },
      },
      wrapperCol: {
        xs: { span: 20 },
        sm: { span: 18 },
      },
      columns,
      data: [],
      editingKey: "",
      cacheData: null,
      dataChangeAble: false,
      interfaceN: "",
      ipN: "",
      maskN: "",
      gatewayN: "",
      scroll:{x: 'max-content' }
    };
  },
  watch: {
    formModel(newVal, oldVal) {
      this.dataChangeAble = false;
      this.data = newVal.data;
      this.cacheData = this.data.map((item) => ({ ...item }));
    },
    cell(newVal, oldVal) {
      this.dataChangeAble = true;
    },
  },
  methods: {
    add() {
      var k = this.data.length;
      var n = {
        key: k,
        interface: this.interfaceN,
        ip: this.ipN,
        mask: this.maskN,
        gateway: this.gatewayN,
      };
      this.data.push(n);
      this.cacheData = this.data.map((item) => ({ ...item }));
      var d = { ports: this.data };
      this.cell.setData(d);
    },
    changeColor(val) {
      this.cell.attr("text/fill", val);
    },
    changeLabel(e) {
      const { value } = e.target;
      this.cell.attr("text/text", value);
      var n = {name : value};
      this.cell.setData(n);
    },
    changeFontSize(val) {
      this.cell.attr("text/fontSize", val);
    },
    changeBackColor(val) {
      this.cell.attr("body/fill", val);
    },
    handleChange(value, key, column) {
      const newData = [...this.data];
      const target = newData.filter((item) => key === item.key)[0];
      if (target) {
        target[column] = value;
        this.data = newData;
      }
    },
    edit(key) {
      const newData = [...this.data];
      const target = newData.filter((item) => key === item.key)[0];
      this.editingKey = key;
      if (target) {
        target.editable = true;
        this.data = newData;
      }
    },
    save(key) {
      const newData = [...this.data];
      const newCacheData = [...this.cacheData];
      const target = newData.filter((item) => key === item.key)[0];
      const targetCache = newCacheData.filter((item) => key === item.key)[0];
      if (target && targetCache) {
        delete target.editable;
        this.data = newData;
        Object.assign(targetCache, target);
        this.cacheData = newCacheData;
      }
      this.editingKey = "";
      var d = { ports: this.data };
      this.cell.setData(d);
    },
    cancel(key) {
      const newData = [...this.data];
      const target = newData.filter((item) => key === item.key)[0];
      this.editingKey = "";
      if (target) {
        Object.assign(
          target,
          this.cacheData.filter((item) => key === item.key)[0]
        );
        delete target.editable;
        this.data = newData;
      }
    },
  },
};
</script>