<template>
  <div
    ref="paste"
    type="text"
    contenteditable="true"
    @paste.stop.prevent="pasteImg($event)"
  >
  </div>
</template>
<script>
    export default {
      name: "imageItem",
      data () {
        return{
            imgData: ""
        }
      },
      methods: {
        clearImg(){
          this.$refs.paste.innerHTML=""
        },
        pasteImg(e) {
          console.log('获得剪切板的内容', e);
          const cbd = e.clipboardData;
          const ua = window.navigator.userAgent;
          // 如果是 Safari 直接 return
          if ( !(e.clipboardData && e.clipboardData.items) ) {
            return ;
          }
          if(cbd.items && cbd.items.length === 2 && cbd.items[0].kind === "string" && cbd.items[1].kind === "file" &&
            cbd.types && cbd.types.length === 2 && cbd.types[0] === "text/plain" && cbd.types[1] === "Files" &&
            ua.match(/Macintosh/i) && Number(ua.match(/Chrome\/(\d{2})/i)[1]) < 49){
            return;
          }
          let vm = this
          for(let i = 0; i < cbd.items.length; i++) {
            let item = cbd.items[i];
            if(item.kind == "file"){
              // blob 就是从剪切板获得的文件，可以进行上传或其他操作
              const blob = item.getAsFile();
              console.log('获得blob', blob);
              if (blob.size === 0) {
                return;
              }

              const reader = new FileReader();
              const imgs = new Image();
              imgs.file = blob;
              reader.onload = (function(aImg) {
                return function(e) {
                  console.log('获得粘贴的结果', e.target.result);
                  aImg.src = e.target.result;
                  alert(e.target.result)
                  vm.$emit("imagedatapost",e.target.result)
                };
              })(imgs);
              reader.readAsDataURL(blob);
              this.$refs.paste.appendChild(imgs);

            }
          }
        }
      }
    }
</script>
<style scoped>
      div{
        width: 100px;
        height: 100px;
        border: grey solid 1px;
        border-radius: 15px;
        overflow: hidden;
      }
</style>