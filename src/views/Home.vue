<template>
  <div class="container">
    <div id="imgArea">
      <div class="box">
        <span @click="addImageFile">添加图片</span>
      </div>
    </div>
    <input class="inputClass" id="imageInput" type="file" multiple/>
     <button class="btn" @click="addImageFile">添加图片</button>
     <button class="btn" @click="upLoad">提交上传</button>
    <span style="fontsize:5px">共{{fileList.length}}个文件</span>
  </div>
</template>
<script>
export default {
  name: "Imageupload",
  data() {
    return {
      baseArray: [],
      fileList: [],
      imgArea: "",
    };
  },
  mounted() {
    let imageInput = document.getElementById("imageInput");
    let imgArea = document.getElementById("imgArea");
    this.imgArea = imgArea;
    imageInput.onchange = (e) => {
      if (imageInput.files.length > 5) {
        alert("最多只能上传5张图片!");
        // imageInput.value=null;//如果设置了样式,隐藏了文件上传的默认的提示,可以不写这一行代码;
        return;
      }
      //当用户初次选择图像时超过了五张终止代码运行;

      for (let k = 0; k < imageInput.files.length; k++) {
        this.fileList.push(imageInput.files[k]);
      }
      //如果用户当初次选择没有超过五张图片,此时就要将input内部的对象列表全部取出放入一个数组当中(fileList)便于后续操作;

      if (this.fileList.length > 5) {
        alert("最多只能上传5张图片!");
        //imageInput.value=null;//如果设置了样式,隐藏了文件上传的默认的提示,可以不写这一行代码;
        this.fileList = this.fileList.slice(0, 5);
      }
      //用户第一次没有添加超过五张图片,但是在第二次添加时超过了五张,则会进入该判断,我们将fileList数组中的对象截取为前五张;

      imgArea.innerHTML = "";
      //先将imaArea这个区域内部的图片都清空;
      //假设用户第一次选两张,fileList里面的数据就交给了map渲染在了浏览器上;
      //但是此时用户如果又新增两张图像进来,fileList内部则会有四张图像,如果我们直接又交给map则会在原有基础上又渲染了之前的图片;
      //所以这里先清空一次Dom,把新的fileList交给map进行渲染即可;
      this.baseArray = this.fileList.map((item) => {
        return new Promise(function (resolve, reject) {
          const fileReader = new FileReader();
          fileReader.readAsDataURL(item);
          fileReader.addEventListener("load", () => {
            let res = fileReader.result;
            if (res != null) {
              resolve(res);
            }
          });
        });
      });
      //将fileList文件数组进行map遍历,因为fileReader是异步的读取,所以使用Promise,最后返回到baseArrey这个数组当中;
      console.log("输出所有的Promise状态的base64图像", this.baseArray);
      Promise.all(this.baseArray).then((res) => {
        this.baseArray = res;
        console.log("输出全部的base64数据", res);
         for (let i = 0; i < this.baseArray.length; i++) {
         var dom = document.createElement("img");
            dom.style.width = "100px";
            dom.style.height = "100px";
            dom.style.marginLeft = "10px";
            dom.setAttribute("src", res[i]);
            dom.classList.add(`box${i}`);
            dom.setAttribute("data-id", i);
            imgArea.appendChild(dom);
        }
        for (var i = 0; i < this.baseArray.length; i++) {
          document.querySelector(`.box${i}`)
            .addEventListener(
              "click",(e)=>{
                  e.stopPropagation();
                  this.render(e.target.dataset.id);
              }
            );
        }
        //给每一项img设置一个click监听事件;点击后将自定义属性传给render方法;
      });
      //把this.baseArray中的所有Promise对象放进Promise.all()中一起获取处理,调用.then()即可获得一个处理完成的base64图像数组;
      //然后创建图像dom元素,设置图像样式,绑定base64的src资源,然后把图像dom元素添加到imgArea当中即可;
    };
  },
  methods: {
    addImageFile() {
      let imageInput = document.getElementById("imageInput");
      imageInput.click();
    },
    render(id) {
      this.baseArray.splice(Number(id), 1);
      this.fileList.splice(Number(id), 1);
      //根据点击传入的索引属性将源数据进行修改;

      this.imgArea.innerHTML = "";
      //清空渲染区域;

      this.baseArray = this.fileList.map((item) => {
        return new Promise(function (resolve, reject) {
          const fileReader = new FileReader();
          fileReader.readAsDataURL(item);
          fileReader.addEventListener("load", () => {
            let res = fileReader.result;
            if (res != null) {
              resolve(res);
            }
          });
        });
      });
      Promise.all(this.baseArray).then((res) => {
        this.baseArray = res;
        console.log("输出全部的base64数据", res);
        for (let i = 0; i < this.baseArray.length; i++) {
         var dom = document.createElement("img");
            dom.style.width = "100px";
            dom.style.height = "100px";
            dom.style.marginLeft = "10px";
            dom.setAttribute("src", res[i]);
            dom.classList.add(`box${i}`);
            //动态追加class属性;
            dom.setAttribute("data-id", i);
            imgArea.appendChild(dom);
        }
        for (var i = 0; i < this.baseArray.length; i++) {
          document.querySelector(`.box${i}`)
            .addEventListener(
              "click",(e)=>{
                  e.stopPropagation();
                  this.render(e.target.dataset.id);
              }
            );
        }
        //重新渲染的imgdom也要设置一个click监听事件;点击后将自定义属性传给render方法;
      });
    },
    upLoad() {
      console.log("需要上传的数据", this.fileList);
    },
  },
  watch: {
      fileList(){
      if(this.fileList.length==0){
        window.location.reload()
      }
    }
  }
};
</script>
<style lang="scss" scoped>
.inputClass[type="file"] {
  color: transparent;
  background: black;
  height: 1px;
  width: 1px;
}
.inputClass {
  width: 70px;
}
.btn {
  background: black;
  color: white;
}
.box {
  width: 100px;
  height: 100px;
  background: rgb(58, 58, 58);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
  color: white;
  margin-bottom: 10px;
}
</style>