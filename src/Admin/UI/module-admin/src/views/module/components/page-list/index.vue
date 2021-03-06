<template>
  <nm-dialog v-bind="dialog" :visible.sync="visible_">
    <template v-slot:title> 页面列表({{ module.name }}) </template>
    <div class="nm-module-page-list">
      <section class="left">
        <el-tabs ref="pageTab" v-model="pageTab" tab-position="left" style="height: 100%;">
          <el-tab-pane v-for="page in pages" :key="page.code" :name="page.code" :label="page.name">
            <span slot="label"><nm-icon :name="page.icon" /> {{ page.name }}</span>
          </el-tab-pane>
        </el-tabs>
      </section>
      <section class="right">
        <nm-tabs>
          <el-tabs type="card">
            <el-tab-pane>
              <span slot="label"><i class="el-icon-date"></i> 详情</span>
              <nm-scrollbar style="height:100%">
                <el-divider content-position="left">页面信息</el-divider>
                <el-form>
                  <el-row>
                    <el-col :span="10" :offset="1">
                      <el-form-item label="标题：">
                        <span>{{ page.name }} </span>
                      </el-form-item>
                    </el-col>
                    <el-col :span="10">
                      <el-form-item label="图标：">
                        <nm-icon class="nm-size-25" :name="page.icon" />
                      </el-form-item>
                    </el-col>
                  </el-row>
                  <el-row>
                    <el-col :span="10" :offset="1">
                      <el-form-item label="路由名称：">
                        <span>{{ page.code }} </span>
                      </el-form-item>
                    </el-col>
                    <el-col :span="10">
                      <el-form-item label="路由路径：">
                        <span>{{ page.path }} </span>
                      </el-form-item>
                    </el-col>
                  </el-row>
                  <el-row>
                    <el-col :span="10" :offset="1">
                      <el-form-item label="路由缓存：">
                        <span><el-switch :value="page.cache" disabled> </el-switch> </span>
                      </el-form-item>
                    </el-col>
                    <el-col :span="10">
                      <el-form-item label="框架内：">
                        <span><el-switch :value="page.frameIn" disabled> </el-switch> </span>
                      </el-form-item>
                    </el-col>
                  </el-row>
                  <el-row>
                    <el-col :span="10" :offset="1">
                      <el-form-item label="权限验证：">
                        <span><el-switch :value="!page.noPermission" disabled> </el-switch> </span>
                      </el-form-item>
                    </el-col>
                  </el-row>
                </el-form>
                <el-divider content-position="left">关联权限</el-divider>
                <el-table :data="pagePermissions" border stripe size="mini" style="width: 100%">
                  <el-table-column prop="name" label="名称" width="180" align="center"> </el-table-column>
                  <el-table-column prop="controller" label="控制器" width="120" align="center"> </el-table-column>
                  <el-table-column prop="action" label="方法" align="center"> </el-table-column>
                  <el-table-column prop="httpMethod" label="请求方式" width="70" align="center">
                    <template slot-scope="scope">
                      {{ getHttpMethodName(scope.row.httpMethod) }}
                    </template>
                  </el-table-column>
                  <el-table-column prop="code" label="编码" align="center" show-overflow-tooltip> </el-table-column>
                </el-table>
              </nm-scrollbar>
            </el-tab-pane>
            <el-tab-pane>
              <span slot="label"><i class="el-icon-date"></i> 按钮</span>
              <nm-box page>
                <el-table :data="buttons" border stripe size="mini" style="width: 100%;">
                  <el-table-column prop="text" label="名称" align="center" width="100"> </el-table-column>
                  <el-table-column prop="icon" label="图标" align="center" width="100">
                    <template v-slot="scope">
                      <nm-icon v-if="scope.row.icon" :name="scope.row.icon" class="nm-size-18" />
                    </template>
                  </el-table-column>
                  <el-table-column prop="code" label="编码" align="center"> </el-table-column>
                  <el-table-column prop="permissions" label="关联权限" align="center">
                    <template v-slot="scope">
                      <p v-for="p in scope.row.permissions" :key="p">{{ p }}</p>
                    </template>
                  </el-table-column>
                </el-table>
              </nm-box>
            </el-tab-pane>
          </el-tabs>
        </nm-tabs>
      </section>
    </div>
  </nm-dialog>
</template>
<script>
import { mapState } from 'vuex'
import { mixins } from 'netmodular-ui'

const { queryByCodes } = $api.admin.permission

export default {
  mixins: [mixins.visible],
  data() {
    return {
      dialog: {
        icon: 'permission',
        width: '900px',
        height: '80%',
        noScrollbar: true
      },
      pageTab: '',
      page: {
        name: '',
        code: '',
        icon: '',
        path: '',
        permissions: [],
        buttons: []
      },
      pagePermissions: [],
      buttons: []
    }
  },
  props: {
    module: Object
  },
  computed: {
    ...mapState('app/page', { allPages: s => s.pages }),
    pages() {
      if (!this.module.code) return []
      return this.allPages.filter(m => m.moduleCode.toLowerCase() === this.module.code.toLowerCase())
    }
  },
  methods: {
    getHttpMethodName(httpMethod) {
      switch (httpMethod) {
        case 0:
          return 'GET'
        case 1:
          return 'PUT'
        case 2:
          return 'POST'
        case 3:
          return 'DELETE'
        case 4:
          return 'HEAD'
        case 5:
          return 'OPTIONS'
        case 6:
          return 'TRACE'
        case 7:
          return 'PATCH'
      }
    }
  },
  watch: {
    pages(val) {
      if (val.length > 0) {
        this.pageTab = val[0].code
      } else {
        this.pageTab = ''
      }
    },
    pageTab(val) {
      this.pagePermissions = []
      this.buttons = []
      if (val) this.page = this.pages.find(m => m.code === val)
      if (!this.page) {
        this.page = {
          name: '',
          code: '',
          icon: '',
          permissions: [],
          buttons: []
        }
      } else {
        queryByCodes(this.page.permissions).then(data => {
          if (data) {
            this.pagePermissions = data
          }
        })

        if (this.page.buttons) {
          for (let key in this.page.buttons) {
            this.buttons.push(this.page.buttons[key])
          }
        }
      }
    }
  }
}
</script>
<style lang="scss">
.nm-module-page-list {
  display: flex;
  flex-direction: row;
  align-items: stretch;
  height: 100%;

  .left {
    flex-shrink: 0;
    .el-tabs__header {
      margin-right: 0;
    }
    .el-tabs__item {
      text-align: left !important;
    }
  }
  .right {
    flex-grow: 1;

    .el-tab-pane {
      box-sizing: border-box;

      .el-scrollbar__view {
        padding: 10px 20px;
      }
    }
  }
}
.left-pane {
  .el-tabs__content {
    display: none;
  }
}
</style>
