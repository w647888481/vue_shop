<template>
  <div>
    <div class="welcome">
      <span>Welcome</span>
    </div>
  </div>
</template>

<script>
export default {}
</script>

<style lang="less" scoped>
.welcome {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 160px;
  height: 90px;
  text-align: center;
  border-radius: 10px;
  background: linear-gradient(145deg, #4286c6, #7075bf);
  box-shadow: 6px 6px 14px #cacaca, -6px -6px 14px #f6f6f6;
  > span {
    line-height: 90px;
    font-size: 30px;
  }
}
</style>
