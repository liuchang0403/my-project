<template>
  <div class="push">
    <!-- 物品标题描述 -->
    <van-cell-group :border="false">
      <van-field
        v-model="title"
        rows="1"
        autosize
        type="content"
        maxlength="10"
        placeholder="请输入物品标题"
        show-word-limit
      />
      <van-field
        v-model="content"
        rows="4"
        autosize
        type="textarea"
        placeholder="请输入品牌型号，新旧程度，入手渠道，转手原因..."
        maxlength="50"
        show-word-limit
      />
    </van-cell-group>
    <!-- Uploader 文件图片上传 子组件-->
    <uploader @func="getFileList"></uploader>
    <!-- Divider 分割线 -->
    <van-divider />
    <!-- 分类 价格 Cell 单元格 -->
    <van-cell-group>
      <van-cell title="分类" :value="sort" icon="coupon-o" is-link center @click="sortShow" />
      <van-cell title="价格" :value="price" icon="gold-coin-o" is-link center @click="getPrice" />
    </van-cell-group>
    <van-divider />
    <!-- Button 发布按钮 -->
    <van-button
      round
      size="large"
      color="linear-gradient(to right, #4bb0ff, #6149f6)"
      @click="pubish"
    >发布</van-button>
    <!-- 分类选择弹出层 -->
    <van-popup v-model="picker_show" position="bottom">
      <van-picker show-toolbar :columns="columns" @cancel="picker_show = false" @confirm="getSort" />
    </van-popup>
    <!-- 价格键盘输入弹出层 -->
    <van-popup v-model="price_show" position="bottom" round :style="{ height: '52%' }">
      <van-field
        class="inprice"
        label="价格"
        placeholder="出售价格"
        :value="new_price"
        @click="NumKey1=true,NumKey2=false"
        @blur="price=new_price"
        @input="price=new_price"
        :style="{readonly:'readonly'}"
        readonly
      />
      <van-field
        class="inprice"
        label="定价"
        placeholder="入手价格"
        :value="old_price"
        @click="NumKey2=true,NumKey1=false"
        readonly
      />
      <!-- 是否全新复选框 -->
      <van-checkbox class="tagCheckbox" v-model="tagChecked" shape="square">全新</van-checkbox>
      <!-- 数字键盘 -->
      <van-number-keyboard
        v-model="new_price"
        :show="NumKey1"
        :maxlength="8"
        theme="custom"
        extra-key="."
        close-button-text="完成"
        @close="price_show =NumKey1= false,price=new_price===''?'开个价':new_price"
        :transition="false"
      />
      <van-number-keyboard
        v-model="old_price"
        :show="NumKey2"
        :maxlength="8"
        theme="custom"
        extra-key="."
        close-button-text="完成"
        @close="price_show =NumKey2= false,price=new_price===''?'开个价':new_price"
        :transition="false"
      />
    </van-popup>
  </div>
</template>

<script>
// 文件上传子组件
import uploader from "@/components/subcomponents/Uploader";

export default {
  data() {
    return {
      // 输入框
      title: "",
      content: "",
      // 类别
      sort: "分个类",
      picker_show: false,
      columns: ["服装", "数码", "运动", "图书"],
      // 价格
      price_show: false,
      NumKey1: false,
      NumKey2: false,
      new_price: "",
      old_price: "",
      price: "开个价",
      // 是否全新
      tagChecked: false,
      // 发布物品随机码id
      pushGoodId: Math.floor(Math.random() * 100000 + 1).toString(),
      // 上传图片文件数组
      fileList: []
    };
  },
  methods: {
    // 分类弹出层显示
    sortShow() {
      this.picker_show = true;
    },
    // 获取分类选择
    getSort(value) {
      this.sort = value;
      this.picker_show = false;
    },
    // 获取价格
    getPrice() {
      this.price_show = true;
      this.NumKey1 = true;
      // 取消手机键盘弹出
      for (
        let i = 0;
        i < document.getElementsByClassName("inprice").length;
        i++
      ) {
        document
          .getElementsByClassName("inprice")
          [i].getElementsByTagName("INPUT")
          .focus(() => {
            document.activeElement.blur();
          });
      }
    },
    // 获取文件上传子组件图片数组
    getFileList(data) {
      // fileList为[]才赋值
      this.fileList = this.fileList ? data : [];
      // console.log(this.fileList);
    },
    // 发布点击事件,添加物品请求
    pubish() {
      if (
        this.title === "" ||
        this.content === "" ||
        this.sort === "分个类" ||
        this.new_price === "" ||
        this.old_price === ""
      ) {
        this.$notify({ type: "warning", message: "请完善物品信息！" });
      } else if (this.fileList.length < 2) {
        this.$notify({ type: "warning", message: "至少添加2张图片！" });
      } else {
        // 发送请求
        this.uploadImg();
      }
    },
    // 上传图片请求
    uploadImg() {
      let formData = new FormData();
      for (let i = 0; i < this.fileList.length; i++) {
        formData.append("files", this.fileList[i]);
      }
      // console.log(formData.getAll("files"));
      this.$axios
        .post("/goods/upload", formData, {
          "Content-Type": "multipart/form-data"
        })
        .then(res => {
          if (res.data.status === "0") {
            console.log("上传图片成功" + res.data.status);
            // console.log(res.data.result);
            // 上传物品信息请求
            this.addToGood(res.data.result);
          }
        })
        .catch(error => {
          console.log(error);
        });
    },
    // 上传物品信息请求
    addToGood(url) {
      let that = this;
      that.$axios
        .post("/goods/add", {
          goodId: that.pushGoodId,
          goodSort: that.sort,
          title: that.title,
          content: that.content,
          new_price: that.returnFloat(that.new_price),
          old_price: that.returnFloat(that.old_price),
          img_url: url,
          seller_name: that.$store.state.user_name,
          seller_img: "https://img.yzcdn.cn/vant/cat.jpeg",
          tag: that.tagChecked ? "全新" : ""
        })
        .then(res => {
          if (res.data.status === "0") {
            that.$notify({ type: "success", message: "发布成功" });
            that.title = this.content = "";
            that.tagChecked = false;
            // 发布成功，添加到用户发布列表
            this.addPublishList();
          } else {
            that.$notify({ type: "warning", message: "用户未登录" });
          }
          // console.log(res);
        })
        .catch(error => {
          console.log(error);
        });
    },
    // 对输入的价格操作函数-数字保留两位小数不足2位自动补零
    returnFloat(value) {
      var value = Math.round(parseFloat(value) * 100) / 100;
      var xsd = value.toString().split(".");
      if (xsd.length == 1) {
        value = value.toString() + ".00";
        return value;
      }
      if (xsd.length > 1) {
        if (xsd[1].length < 2) {
          value = value.toString() + "0";
        }
        return value;
      }
    },
    addPublishList() {
      let that = this;
      that.$axios
        .post("/users/publish/add", {
          userId: that.$store.state.user_id,
          goodId: that.pushGoodId,
          date: Date()
        })
        .then(res => {
          if (res.data.status === "0") {
            console.log("添加成功" + res.data.status);
          }
        })
        .catch(error => {
          console.log(error);
        });
    }
  },
  //注册子组件，文经上传
  components: {
    uploader
  }
};
</script>
<style lang="scss" scoped>
.push {
  margin-left: 0.5rem;
  margin-right: 0.5rem;
}
.tagCheckbox {
  margin: 0.7rem;
  display: flex;
  justify-content: flex-end;
}
</style>