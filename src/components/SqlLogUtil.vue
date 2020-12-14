<template>
  <div>
    <h1>SQL提取工具</h1>
    <div style="margin: 20px">
      <textarea v-model="log" rows="10" cols="100" style="resize:none" placeholder="在此处粘贴原始SQL日志" />
    </div>
    <button type="button" @click="getSql">提取</button>
    <div style="margin: 20px">
      <textarea v-model="sql" rows="10" cols="100" style="resize:none" placeholder="提取的SQL将显示在此处" readonly="true" />
    </div>
  </div>
</template>

<script>
export default {
  name: 'SqlLogUtil',
  data () {
    return {
      log: '',
      sql: ''
    }
  },
  methods: {
    getSql () {
      let lines = this.log.split('\n')
      let preparing = null
      let parameters = null
      
      for (const line of lines) {
        if (line.indexOf('==>  Preparing: ') >= 0) {
          preparing = line.substr(line.indexOf('==>  Preparing: ') + 16)
        }
        if (line.indexOf('==> Parameters: ') >= 0) {
          parameters = line.substr(line.indexOf('==> Parameters: ') + 16)
        }
        if (preparing != null && parameters != null) {
          break
        }
      }
      const paramsArray = parameters.split(',')
      if (preparing.match(/\?/g).length !== paramsArray.length) {
        alert('sql参数数目有误')
        return
      }
      let sql = preparing
      for (let param of paramsArray) {
        //去除空格
        param = param.trim()
        if (param === 'null') {
          sql = sql.replace(/\?/, param)
        } else {
          const type = param.match(/\(\w+\)$/)[0]
          const realParam = param.substr(0, param.length - type.length)
          switch (type.substr(1, type.length - 2).toLowerCase()) {
            case 'string':
            case 'date':
            case 'time':
            case 'timestamp':
              sql = sql.replace(/\?/, '\'' + realParam + '\'')
              break
            default:
              sql = sql.replace(/\?/, realParam)
              break
          }
        }
      }
      console.log(sql)
      this.sql = sql
    }
  }
}
</script>