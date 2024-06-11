<template>
  <div>
    <div class="grid grid-cols-6 h-screen bg-white">
      <!-- 左边栏 -->
      <div class="col-span-6 md:col-span-3 sm:col-span-6">
        <div
          class="login-container-left flex justify-center items-center flex-col"
        >
          <div
            class="animate__animated animate__bounceInLeft items-center flex flex-col"
          >
            <h2 class="font-bold text-4xl mb-7 text-white">
              PersonYun 个人博客
            </h2>
            <p class="text-white" style="margin-left: 20px">
              祝你不用奔赴大海，也能春暖花开。祝你不用颠沛流离，也能遇到陪伴。<br />祝你不用熬过黑夜，也能等到晚安。
            </p>
            <img src="@/assets/developer.png" class="login-image" />
          </div>
        </div>
      </div>
      <!-- 右边栏 -->
      <div class="col-span-6 md:col-span-3 sm:col-span-6">
        <div
          class="login-container-right flex justify-center items-center flex-col animate__animated animate__bounceInRight animate__fast"
        >
          <h2 class="font-bold text-3xl text-gray-100 mt-5">欢迎回来!</h2>
          <div
            class="flex items-center justify-center my-5 text-gray-400 space-x-2"
          >
            <span class="h-[1px] w-16 bg-gray-200"></span>
            <span>账号密码登录</span>
            <span class="h-[1px] w-16 bg-gray-200"></span>
          </div>
          <div>
            <el-form
              ref="formRef"
              :rules="rules"
              :model="form"
              class="w-[300px]"
            >
              <el-form-item prop="username">
                <el-input
                  v-model="form.username"
                  :prefix-icon="User"
                  placeholder="请输入用户名"
                  size="large"
                  clearable
                />
              </el-form-item>
              <el-form-item prop="password">
                <el-input
                  v-model="form.password"
                  type="password"
                  autocomplete="off"
                  :prefix-icon="Lock"
                  placeholder="请输入密码"
                  show-password
                  size="large"
                  clearable
                />
              </el-form-item>
              <el-form-item>
                <el-input
                  v-model="form.canvas"
                  autocomplete="off"
                  placeholder="请输入验证码"
                  size="large"
                  clearable
                  class="yanzhengma"
                />
                <canvas id="canvas" @click="handleCanvas"> </canvas>
              </el-form-item>
              <el-form-item>
                <el-button
                  round
                  type="primary"
                  @click="onSubmit"
                  :loading="loading"
                  class="ml-15 login-btn mt-4"
                  size="large"
                >
                  登 录
                </el-button>
                <el-button
                  round
                  @click="onregistered"
                  :loading="loading"
                  class="ml-10 login-btn mt-4"
                  size="large"
                >
                  注 册
                </el-button>
              </el-form-item>
            </el-form>
          </div>
        </div>
      </div>
    </div>
    <!-- 注册 -->
    <el-dialog v-model="dialogFormVisible" title="注册" width="400" center>
      <el-form ref="regformRef" :rules="regrules" :model="regform">
        <el-form-item
          label="用户名"
          :label-width="formLabelWidth"
          prop="username"
        >
          <el-input v-model="regform.username" autocomplete="off" />
        </el-form-item>
        <el-form-item label="密 码" :label-width="65" prop="password">
          <el-input v-model="regform.password" autocomplete="off" />
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="regSubmit">确定</el-button>
          <el-button @click="regquxiao(regformRef)"> 取消 </el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>
  
<script setup>
import { reactive, ref, onMounted, onBeforeUnmount } from "vue";
import { login, register } from "@/api/admin/user";
import { showMessage } from "@/composables/util";
import { useRouter } from "vue-router";
import { useStore } from "vuex";
import { setToken } from "@/composables/auth";
import { User, Lock } from "@element-plus/icons-vue";

const yanzhen = ref(""); //输入框双向绑定的值
const true_code = ref(""); //保存正确的验证码
const yanzhen_arr = ref([]); //只用于传参，并且数组长度不能「多于」下面验证码遍历的次数，不然最终得到的true_code会有问题
//比如下面是4个验证码，可以是[1,2,3,4]及以下，但是不能是[1，2，3，4，5]， 因为5无法被替换

const router = useRouter();
const store = useStore();

const form = reactive({
  username: "",
  password: "",
  canvas: "",
});

const rules = {
  username: [
    {
      required: true,
      //  min: 6, max: 18,
      message: "用户名为4~16字符之间（中文、字母、数字或下划线）",
      trigger: "blur",
    },
  ],
  password: [
    {
      required: true,
      // min: 6, max: 18,
      message: "密码为6~18位字母、数字和符号",
      trigger: "blur",
    },
  ],
};
const nums = ref([]);
const formRef = ref(null);
const loading = ref(false);
//登录
const onSubmit = () => {
  const aa = nums.value.join("");
  if (!form.canvas) {
    ElMessage({ type: "error", message: "验证码不能为空！" });
    return;
  }
  if (form.canvas != aa) {
    ElMessage({ type: "error", message: "验证码错误！" });
    handleCanvas();
    return;
  }
  // 登录表单验证
  formRef.value.validate((valid) => {
    if (!valid) {
      return false;
    }
    loading.value = true;
    login(form.username, form.password)
      .then((res) => {
        if (res.success == true) {
          // 提示成功
          let message = res.message;
          showMessage("登录成功", "success");
          // notification('登录成功')

          let token = res.data.token;
          // 存储 token
          setToken(token);

          // 跳转到后台页面
          router.push("/admin");
        } else {
          let message = res.message;
          showMessage(message, "error");
        }
      })
      .finally(() => {
        loading.value = false;
      });
  });
};
//显示验证码区域内容信息
const draw = (show_num) => {
  var canvas_width = document.querySelector("#canvas").clientWidth;
  var canvas_height = document.querySelector("#canvas").clientHeight;
  var canvas = document.getElementById("canvas"); //获取到canvas
  var context = canvas.getContext("2d"); //获取到canvas画图
  canvas.width = canvas_width;
  canvas.height = canvas_height;
  var sCode =
    "a,b,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z,A,B,C,E,F,G,H,J,K,L,M,N,P,Q,R,S,T,W,X,Y,Z,1,2,3,4,5,6,7,8,9,0";
  var aCode = sCode.split(",");
  var aLength = aCode.length; //获取到数组的长度

  //4个验证码数
  for (var i = 0; i <= 3; i++) {
    var j = Math.floor(Math.random() * aLength); //获取到随机的索引值
    var deg = (Math.random() * 30 * Math.PI) / 180; //产生0~30之间的随机弧度
    var txt = aCode[j]; //得到随机的一个内容
    show_num[i] = txt.toLowerCase(); // 依次把取得的内容放到数组里面
    var x = 10 + i * 20; //文字在canvas上的x坐标
    var y = 20 + Math.random() * 8; //文字在canvas上的y坐标
    context.font = "bold 23px 微软雅黑";

    context.translate(x, y);
    context.rotate(deg);

    context.fillStyle = randomColor();
    context.fillText(txt, 0, 0);

    context.rotate(-deg);
    context.translate(-x, -y);
  }
  //验证码上显示6条线条
  for (var i = 0; i <= 5; i++) {
    context.strokeStyle = randomColor();
    context.beginPath();
    context.moveTo(Math.random() * canvas_width, Math.random() * canvas_height);
    context.lineTo(Math.random() * canvas_width, Math.random() * canvas_height);
    context.stroke();
  }
  //验证码上显示31个小点
  for (var i = 0; i <= 30; i++) {
    context.strokeStyle = randomColor();
    context.beginPath();
    var x = Math.random() * canvas_width;
    var y = Math.random() * canvas_height;
    context.moveTo(x, y);
    context.lineTo(x + 1, y + 1);
    context.stroke();
  }
  nums.value = [];
  for (let i = 0; i < 4; i++) {
    nums.value.push(show_num[i]); // 将 show_num 的元素添加到 nums 数组中
  }
};
//得到随机的颜色值
const randomColor = () => {
  var r = Math.floor(Math.random() * 256);
  var g = Math.floor(Math.random() * 256);
  var b = Math.floor(Math.random() * 256);
  return "rgb(" + r + "," + g + "," + b + ")";
};
const handleCanvas = () => {
  draw(yanzhen_arr);
};
//注册
const dialogFormVisible = ref(false);
const onregistered = () => {
  dialogFormVisible.value = true;
};
const regform = reactive({
  username: "",
  password: "",
});
const regrules = reactive({
  username: [
    { required: true, message: "用户名为4~16位之间字符", trigger: "blur" },
    {
      min: 4,
      max: 16,
      message: "4~16位中文、字母、数字或下划线",
      trigger: "blur",
    },
  ],
  password: [
    {
      required: true,
      message: "密码为6~18位字符，必须包含字母、数字和符号",
      min: 6,
      max: 18,
      trigger: "blur",
    },
  ],
});
const regformRef = ref(null);
const regSubmit = (formEl) => {
  regformRef.value.validate((valid) => {
    if (!valid) {
      return;
    }
  });
  register(regform).then((res) => {
    showMessage("注册成功", "success");
  });
  dialogFormVisible.value = false;
  formEl.resetFields();
};
//取消
const regquxiao = (formEl) => {
  if (!formEl) return;
  formEl.resetFields();
  dialogFormVisible.value = false;
};
function onKeyUp(e) {
  if (e.key == "Enter") {
    onSubmit();
  }
}

// 添加键盘监听
onMounted(() => {
  document.addEventListener("keyup", onKeyUp);
  handleCanvas();
});

// 移除键盘监听
onBeforeUnmount(() => {
  document.removeEventListener("keyup", onKeyUp);
});
</script>

<style>
#canvas {
  margin-left: 10px;
  width: 113px;
  height: 40px;
  float: right;
  margin-right: 1%;
  display: block;
  border: 1px solid #eaefea;
  border-radius: 5px;
  cursor: pointer;
  /* background-color: red; */
}
.yanzhengma {
  width: 174px;
}
:deep([type="text"]:focus) {
  border-color: transparent !important;
}

.login-container {
  height: 100vh;
  width: 100%;
  background-color: #fff;
}

.login-container-left {
  height: 100%;
  background: #001428;
  color: #fff;
}

.login-container-right {
  background: #001428;
  height: 100%;
}

.login-image {
  max-width: 500px;
  height: auto;
  /* height: 600px; */
}
</style>
  