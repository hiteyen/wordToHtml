<template>
  <div class="home">
    <input
      ref="upload"
      type="file"
      accept=".docx"
      @change="readFileAsArrayBuffer"
    />
  </div>
</template>

<script>
import axios from "axios";
const mammoth = require("mammoth");
export default {
  name: "Home",
  components: {},
  data() {
    return {};
  },
  methods: {
    // 监听input change事件，读取文件转ArrayBuffer
    readFileAsArrayBuffer(e) {
      const that = this;
      const file = e.target.files[0];
      const reader = new FileReader();

      reader.readAsArrayBuffer(file);

      reader.onload = function(event) {
        const arrayBuffer = event.target["result"];
        that.wordToHtml(arrayBuffer);
      };
    },
    // base64 转 blob, mime为类型，base64为,后的内容
    base64ToBlob: (base64, mimeString) => {
      const byteString = window.atob(base64);
      const bufferObj = new ArrayBuffer(byteString.length);
      const uintObj = new Uint8Array(bufferObj);
      for (let i = 0; i < byteString.length; i++) {
        uintObj[i] = byteString.charCodeAt(i);
      }
      return new Blob([uintObj], { type: mimeString });
    },
    // word转HTML
    wordToHtml(arrayBuffer) {
      const that = this;
      // 转换配置，把base64图片转二进制在上传服务器
      const options = {
        convertImage: mammoth.images.imgElement(image => {
          return image.read("base64").then(async imageBuffer => {
            const res = await that.uploadImage(imageBuffer, image.contentType);
            return {
              src: res.data.data
            };
          });
        })
      };
      mammoth.convertToHtml({ arrayBuffer }, options).then(docx => {
        console.log("最终结果：", docx);
      });
    },
    // 上传图片
    async uploadImage(base64Image, type) {
      const that = this;
      const formData = new FormData();
      formData.append("file", that.base64ToBlob(base64Image, type));
      return await axios({
        method: "post",
        url: "/api/api/base-service/file/uploadFile",
        data: formData,
        config: { headers: { "Content-Type": "multipart/form-data" } }
      });
    }
  }
};
</script>
