<template>
    <div>
        Home组件
        <el-button type="info" @click="logout">退出</el-button>

    </div>
</template>
<script>
export default {
    methods: {
        logout () {
            /* 清空token信息 */
            window.sessionStorage.clear()
            this.$router.push('/login')
            console.log('jasd')
        }
    }
}
</script>
<style lang="less" scoped>

</style>
