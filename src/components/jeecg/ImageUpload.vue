<template>
  <div class="clearfix">
    <a-upload
      action="http://192.168.1.114:8080/product/product/add"
      list-type="picture-card"
      :file-list="fileList"
      @preview="handlePreview"
      @change="handleChange"
      @click="getcbPic">
      <div v-if="fileList.length < 8" >
        <a-icon type="plus" @click="getcbPic"/>
        <div class="ant-upload-text">
          Upload
        </div>
      </div>
    </a-upload>
    <a-modal :visible="previewVisible" :footer="null" @cancel="handleCancel">
      <img alt="example" style="width: 100%" :src="previewImage" />
    </a-modal>
  </div>
</template>
<script>
  import * as evt from "codemirror";

  function getBase64(file) {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.readAsDataURL(file);
      console.dir(reader.result);
      reader.onload = () => resolve(reader.result);
      reader.onerror = error => reject(error);
    });
  }
  export default {
    data() {
      return {
        name: "ImageUpload",
        previewVisible: false,
        previewImage: '',
        dataI: "",
        fileList: [
          // {
          //   uid: '-1',
          //   name: 'image.png',
          //   status: 'done',
          //   url: 'https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png',
          // }

        ],
      };
    },
    methods: {
      handleCancel() {
        this.previewVisible = false;
      },
      async handlePreview(file) {
        if (!file.url && !file.preview) {
          file.preview = await getBase64(file.originFileObj);
          this.dataI = await getBase64(file.originFileObj);
        }
        this.previewImage = file.url || file.preview;
        this.previewVisible = true;
      },
      handleChange({ fileList }) {
        this.fileList = fileList;
      },
      getcbPic(){
        // let clipdata = evt.clipboardData || window.Clipboard().data();
        navigator.permissions.query({name: "clipboard-read"}).then(result => {
          if (result.state == "granted" || result.state == "prompt") {
            navigator.clipboard.read().then(data => {
              console.dir(data[0])
              // for (let i=0; i<data.items.length; i++) {
                if (data[0].type != "image/jpg") {
                  const blob = data[0].getType("image/jpg");
                  alert("Clipboard contains non-image data. Unable to access it.");
                } else {
                  const blob = data[0].getType("image/jpg");
                  // imgElem.src = URL.createObjectURL(blob);
                }
              // }
            });
          }
        });
      }
    },
  };
</script>
<style>
  /* you can make up upload button and sample style by using stylesheets */
  .ant-upload-select-picture-card i {
    font-size: 32px;
    color: #999;
  }

  .ant-upload-select-picture-card .ant-upload-text {
    margin-top: 8px;
    color: #666;
  }
</style>
