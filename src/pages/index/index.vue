<template>
   <view class="index">
    <view>
       <PhotoTemple  v-if="cavnasShow" />
    </view>
    <view class="btns"  @tap="cavnasShow = true"  >
      生成带二维码海报
    </view>
  </view>
</template>

<script>
import PhotoTemple from '../../component/photoTemple'
import './index.scss'

export default {
  components:{
    PhotoTemple
  },
  data () {
    return {
      msg: 'Hello world!',
      cavnasShow:false,
    }
  }
}
</script>

<style lang="scss">
.index{
   position: fixed;
   left:0;
   right: 0;
   bottom: 0;
   top:0;
   background: #000;
   .btns{
      width: 400px;
      height: 80px;
      background: #FF4006;
      color: #fff;
      border-radius:20px ;
      text-align: center;
      line-height: 80px;
      position: absolute;
      left:50%;
      top:50%;
      transform: translate(-50%,-50%);
   }
}

</style>
