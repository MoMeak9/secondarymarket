<template>
  <el-container id="user">
    <el-header>
      <el-menu default-active="/user" class="el-menu-demo" mode="horizontal" router>
        <el-menu-item index="/">首页</el-menu-item>
        <el-menu-item index="/search">搜索</el-menu-item>
        <el-menu-item index="/user">个人中心</el-menu-item>
      </el-menu>
    </el-header>
    <el-container class="content">
      <div class="container">
        <el-col :span="4" class="ct-left">
          <div class="img-wrap">
            <el-upload
                class="avatar-uploader"
                :headers="headers"
                :show-file-list="false"
                action="https://zhuanxiaoer.cn/market/user/updateUserInfo"
                name="profile"
                :on-success="handleProfilePhotoSuccess"
                :before-upload="beforeProfilePhotoUpload">
              <img v-if="userBean.profileUrl" :src="userBean.profileUrl" class="avatar" style="object-fit:cover" alt="">
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
              <div class="img-bottom">上传头像</div>
            </el-upload>
          </div>
          <div class="menu-tab">
            <div v-for="item in menuTab" :key="item.id" :tabindex="item.id"
                 :class="[ {'isActive':item.isActive}, 'menu-tab-item' ]" @click="toggleMenuItem(item)">{{ item.txt }}
            </div>
          </div>
        </el-col>
        <el-col :span="20" class="ct-right">
          <div class="right-container">
            <!--            用户信息展示-->
            <div class="user-info" v-show="menuTab[0].isActive">
              <!--              基本信息-->
              <myInfo></myInfo>
            </div>
            <!--            购买历史-->
            <div class="history-order" v-show="menuTab[1].isActive">
              <el-table :data="historyOrder" stripe style="width: 100%">
                <el-table-column type="expand">
                  <template slot-scope="props">
                    <el-form label-position="left" inline class="demo-table-expand">
                      <el-form-item label="联系电话">
                        <span>{{ props.row.phone }}</span>
                      </el-form-item>
                      <el-form-item label="收货地址">
                        <span>{{ props.row.address }}</span>
                      </el-form-item>
                      <el-form-item label="订单号">
                        <span>{{ props.row.orderNo }}</span>
                      </el-form-item>
                      <el-form-item label="商品描述">
                        <span>{{ props.row.commDesc }}</span>
                      </el-form-item>
                    </el-form>
                  </template>
                </el-table-column>
                <!--                商品图片-->
                <el-table-column label="图片">
                  <template slot-scope="scope">
                    <el-image :src="scope.row.commPicList[0]" fit='cover' :preview-src-list="scope.row.commPicList"
                              style="width: 50px;height: 50px"></el-image>
                  </template>
                </el-table-column>
                <el-table-column prop="createTime" label="下单时间" width="180"></el-table-column>
                <el-table-column prop="commName" label="商品名称"></el-table-column>
                <el-table-column prop="num" label="数量" width="50"></el-table-column>
                <el-table-column prop="price" label="单价" width="50"></el-table-column>
                <!--                描述-->
                <el-table-column label="状态">
                  <div slot-scope="scope">
                    <div v-if="scope.row.orderStatus === 0">待处理</div>
                    <div v-else-if="scope.row.orderStatus === 1">已发货</div>
                    <div v-else-if="scope.row.orderStatus === 2">已收货</div>
                    <div v-else-if="scope.row.orderStatus === 3">申请取消</div>
                    <div v-else>已取消</div>
                  </div>
                </el-table-column>
                <el-table-column label="操作" width="180">
                  <div slot-scope="scope">
                    <div v-if="scope.row.orderStatus === 0">
                      <el-button type="danger" size="small" @click="changeStatus(scope.row, 3)">取消订单</el-button>
                    </div>
                    <div v-else-if="scope.row.orderStatus === 1">
                      <el-button type="primary" size="small" @click="changeStatus(scope.row, 2)">确认</el-button>
                      <el-button type="danger" size="small" @click="changeStatus(scope.row, 3)">取消订单</el-button>
                    </div>
                    <div v-if="scope.row.orderStatus === 2">
                      <el-tag type="success">订单已完成！</el-tag>
                    </div>
                    <div v-else-if="scope.row.orderStatus === 3">
                      <el-tag>等待确认</el-tag>
                    </div>
                    <div v-else-if="scope.row.orderStatus === 4">
                      <el-tag type="success">订单已取消！</el-tag>
                    </div>
                  </div>
                </el-table-column>
              </el-table>
            </div>
            <!--            我的商品 个人商品发布列表 myCommodity-->
            <div class="my-commodity" v-show="menuTab[2].isActive">
              <el-table :data="myCommodity" stripe style="width: 100%">
                <!--                商品图片-->
                <el-table-column label="图片" width="100">
                  <template slot-scope="scope">
                    <el-image :src="scope.row.commPicList[0]" fit='cover' :preview-src-list="scope.row.commPicList"
                              style="width: 50px;height: 50px"></el-image>
                  </template>
                </el-table-column>
                <el-table-column prop="commodity.commName" label="商品名称" width="100"></el-table-column>
                <el-table-column prop="commodity.commDesc" label="商品描述" width="200"></el-table-column>
                <el-table-column prop="commodity.commSale" label="卖出" width="50"></el-table-column>
                <el-table-column prop="commodity.commStock" label="库存" width="50"></el-table-column>
                <el-table-column prop="commodity.auditStatus" label="审核状态">
                  <div slot-scope="scope">
                    <div v-if="scope.row.commodity.auditStatus === 0">待审核</div>
                    <div v-else-if="scope.row.commodity.auditStatus === 1">审核通过</div>
                    <div v-else>驳回</div>
                  </div>
                </el-table-column>
                <el-table-column label="商品操作">
                  <div slot-scope="scope">
                    <el-button type="primary" size="small" @click="getCommodityInfo(scope.row.commodity.commNo)">查看
                    </el-button>
                    <el-button type="danger" size="small" @click="deleteComm(scope.row.commodity.commNo)">下架商品
                    </el-button>
                  </div>
                </el-table-column>
              </el-table>
            </div>
            <div class="publish" v-show="menuTab[3].isActive">
              <div class="right-item" style="height: 195px; line-height:195px">
                <div class="item-label">图片</div>
                <div class="item-info">
                  <el-upload
                      class="avatar-uploader"
                      :headers="headers"
                      :show-file-list="false"
                      name=""
                      :on-success="handleAvatarSuccess"
                      :before-upload="beforeAvatarUpload">
                    <img v-if="imageUrl" :src="imageUrl" class="avatar">
                    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                  </el-upload>
                </div>
              </div>
              <div class="right-item">
                <div class="item-label">名称</div>
                <div class="item-info" style="width: 500px">
                  <el-input v-model="commodity.name" placeholder="请输入商品名称" maxlength="50" show-word-limit></el-input>
                </div>
              </div>
              <div class="right-item" style="padding-bottom: 7em">
                <div class="item-label">描述</div>
                <div class="item-info" style="width: 500px">
                  <el-input
                      type="textarea"
                      placeholder="请输入商品描述"
                      v-model="commodity.description"
                      :autosize="{ minRows:7, maxRows: 8}"
                      maxlength="150"
                      show-word-limit
                  >
                  </el-input>
                </div>
              </div>
              <div class="right-item">
                <div class="item-label">数量</div>
                <div class="item-info">
                  <el-input-number v-model="commodity.quantity" :min="1" size="small"></el-input-number>
                </div>
              </div>
              <div class="right-item">
                <div class="item-label">单价</div>
                <div class="item-info" style="width:300px">
                  <el-input v-model="commodity.price" placeholder="请输入单件商品价格，单位：元"></el-input>
                </div>
              </div>
              <div class="right-item">
                <div class="item-label">类别</div>
                <el-radio-group v-model="commType" @change="select(commType)">
                  <el-radio-button label="衣物"></el-radio-button>
                  <el-radio-button label="数码"></el-radio-button>
                  <el-radio-button label="食品"></el-radio-button>
                  <el-radio-button label="图书"></el-radio-button>
                  <el-radio-button label="化妆品"></el-radio-button>
                  <el-radio-button label="文具"></el-radio-button>
                  <el-radio-button label="居家"></el-radio-button>
                </el-radio-group>
              </div>
              <div class="right-item">
                <div class="item-label">标签</div>
                <div>
                  <el-tag
                      :key="tag"
                      v-for="tag in commodity.dynamicTags"
                      closable
                      :disable-transitions="false"
                      @close="handleClose(tag)">
                    {{ tag }}
                  </el-tag>
                  <el-input
                      class="input-new-tag"
                      v-if="commodity.inputVisible"
                      v-model="commodity.inputValue"
                      ref="saveTagInput"
                      size="small"
                      @keyup.enter.native="handleInputConfirm"
                      @blur="handleInputConfirm"
                  >
                  </el-input>
                  <el-button v-else class="button-new-tag" size="small" @click="showInput">+ 自定义标签</el-button>
                </div>
              </div>
              <div class="right-item">
                <div class="item-label"></div>
                <div class="item-info">
                  <el-button type="danger" @click="publish()">发布商品</el-button>
                </div>
              </div>
            </div>
            <!--            卖家订单管理-->
            <div class="pending-order" v-show="menuTab[4].isActive">
              <el-table :data="pendingOrder" stripe style="width: 100%">
                <el-table-column type="expand">
                  <template slot-scope="props">
                    <el-form label-position="left" inline class="demo-table-expand">
                      <el-form-item label="收货人">
                        <span>{{ props.row.consignee }}</span>
                      </el-form-item>
                      <el-form-item label="联系电话">
                        <span>{{ props.row.phone }}</span>
                      </el-form-item>
                      <el-form-item label="收货地址">
                        <span>{{ props.row.address }}</span>
                      </el-form-item>
                      <el-form-item label="订单号">
                        <span>{{ props.row.commNo }}</span>
                      </el-form-item>
                      <el-form-item label="送货时间">
                        <span>{{ props.row.deTimeFrom }}至{{ props.row.deTimeTo }}</span>
                      </el-form-item>
                    </el-form>
                  </template>
                </el-table-column>
                <el-table-column label="图片">
                  <template slot-scope="scope">
                    <el-image :src="scope.row.commPicList[0]" fit='cover' :preview-src-list="scope.row.commPicList"
                              style="width: 50px;height: 50px"></el-image>
                  </template>
                </el-table-column>
                <el-table-column prop="commName" label="商品名"></el-table-column>
                <el-table-column prop="createTime" label="下单时间"></el-table-column>
                <el-table-column prop="num" label="数量"></el-table-column>
                <el-table-column label="状态">
                  <div slot-scope="scope">
                    <div v-if="scope.row.orderStatus === 0">待处理</div>
                    <div v-else-if="scope.row.orderStatus === 1">已发货</div>
                    <div v-else-if="scope.row.orderStatus === 2">已确认收货</div>
                    <div v-else-if="scope.row.orderStatus === 3">申请取消</div>
                    <div v-else>已取消</div>
                  </div>
                </el-table-column>
                <el-table-column label="处理" width="180">
                  <div slot-scope="scope">
                    <div v-if="scope.row.orderStatus === 0">
                      <el-button type="primary" size="small" @click="changeStatus(scope.row, 1)">发货</el-button>
                      <el-button type="danger" size="small" @click="changeStatus(scope.row, 4)">取消订单</el-button>
                    </div>
                    <div v-else-if="scope.row.orderStatus === 1">
                      <el-button type="danger" size="small" @click="changeStatus(scope.row,4)">取消订单</el-button>
                    </div>
                    <div v-if="scope.row.orderStatus === 2">
                      <el-tag type="success">订单已完成！</el-tag>
                    </div>
                    <div v-else-if="scope.row.orderStatus === 3">
                      <el-button type="danger" size="small" @click="changeStatus(scope.row,4 )">确认取消</el-button>
                    </div>
                    <div v-else-if="scope.row.orderStatus === 4">
                      <el-tag type="success">订单已取消！</el-tag>
                    </div>
                  </div>
                </el-table-column>
              </el-table>
            </div>
          </div>
        </el-col>
      </div>
    </el-container>
  </el-container>
</template>
<script>
import myInfo from "@/components/myInfo";
import {Server} from "@/service/api";

export default {
  components: {
    myInfo
  },
  data() {
    return {
      commType: '',
      fileList: [],
      headers: {
        Authorization: this.$store.state.token
      },
      token: '',
      userBean: {
        userName: '',
        isBan: '',
        userRoot: '',
        userInfo: '',
        userSex: '',
        profileUrl: '',
        userEmail: '',
        authentication: '',
      },
      menuTab: [
        {
          id: 1,
          txt: '我的资料',
          isActive: true
        },
        {
          id: 2,
          txt: '购买历史',
          isActive: false
        },
        {
          id: 3,
          txt: '我的商品',
          isActive: false
        },
        {
          id: 4,
          txt: '发布商品',
          isActive: false
        },
        {
          id: 5,
          txt: '订单管理',
          isActive: false
        }
      ],
      historyOrder: [],
      myCommodity: [],
      pendingOrder: [],
      commodity: {
        name: '',
        description: '',
        quantity: 1,
        price: '',
        commTag: '',
        dynamicTags: ['包邮'],
        inputVisible: false,
        inputValue: ''
      },
      imageUrl: '',
      imageFile: '',
      AuthenticationImageFile: '',
      socket: null,
    }
  },
  mounted() {
    this.imageFile = new FormData()
    this.userBean = this.$store.state.userBean
    this.token = this.$store.state.token
    this.$nextTick(function () {
      if (this.token === null || this.token === '') {
        this.$notify({
          title: '未登入',
          message: '即将前往登入页',
          type: 'error'
        })
        clearTimeout(this.timer);
        this.timer = setTimeout(() => {
          this.$router.push({path: '/login'});
        }, 1500);
      }
      this.initData()
    })
  },
  methods: {
    //初始化方法
    initData() {
      // 获取购买历史(用户提交的订单列表接口)
      Server.queryUserSubmitOrderList(this.token).then(resp => {
        if (resp.code === 1) {
          this.historyOrder = resp.obj
          for (let i = 0; i < this.historyOrder.length; i++) {
            this.historyOrder[i].createTime = this.rTime(this.historyOrder[i].createTime)
          }
        }
      }).catch(function (error) {
        console.log(error)
      })
      // 获取用户在售卖的商品
      Server.queryUserComm(this.token).then(resp => {
        if (resp.code === 1) {
          this.myCommodity = resp.obj
        }
      }).catch(function (error) {
        console.log(error)
      })
      // 获取用户已卖出的商品
      Server.queryUserReceiveOrderList(this.token).then(resp => {
        if (resp.code === 1) {
          this.pendingOrder = resp.obj
          for (let i = 0; i < this.pendingOrder.length; i++) {
            this.pendingOrder[i].createTime = this.rTime(this.pendingOrder[i].createTime)
            this.pendingOrder[i].deTimeFrom = this.rTime(this.pendingOrder[i].deTimeFrom)
            this.pendingOrder[i].deTimeTo = this.rTime(this.pendingOrder[i].deTimeTo)
          }
        }
      }).catch(function (error) {
        console.log(error)
      })
    },
    //更改订单状态
    changeStatus(row, orderStatus) {
      if (orderStatus === 1) {
        Server.updateOrderStatus({
          orderNo: row.orderNo,
          orderStatus: orderStatus,
        }, this.token).then(resp => {
          if (resp.code === 1) {
            this.$notify({
              title: '成功',
              message: '发货成功！请及时发货！',
              type: 'success'
            })
            this.initData()
          }
        }).catch(function (error) {
          console.log(error)
        })
      } else if (orderStatus === 4) {
        Server.updateOrderStatus({
          orderNo: row.orderNo,
          orderStatus: orderStatus,
        }, this.token).then(resp => {
          if (resp.code === 1) {
            this.$notify({
              title: '成功',
              message: '已取消订单',
              type: 'success'
            })
            this.initData()
          }
        }).catch(function (error) {
          console.log(error)
        })
      } else if (orderStatus === 2) {
        Server.updateOrderStatus({
          orderNo: row.orderNo,
          orderStatus: orderStatus
        }, this.token).then(resp => {
          if (resp.code === 1) {
            this.$notify({
              title: '成功',
              message: '已确认收货',
              type: 'success'
            })
            this.initData()
          }
        }).catch(function (error) {
          console.log(error)
        })
      } else {
        Server.updateOrderStatus({
          orderNo: row.orderNo,
          orderStatus: orderStatus,
        }, this.token).then(resp => {
          if (resp.code === 1) {
            this.$notify({
              title: '成功',
              message: '订单申请取消',
              type: 'success'
            })
            this.initData()
          }
        }).catch(function (error) {
          console.log(error)
        })
      }
    },
    //下架商品commNo
    deleteComm(commNo) {
      Server.deleteCommComment({
        commNo: commNo,
      }, this.token).then(resp => {
        if (resp.code === 1) {
          this.$notify({
            title: '成功',
            message: '操作成功！',
            type: 'success'
          })
          this.initData()
        } else {
          this.$notify({
            title: '失败',
            message: '系统错误！',
            type: 'error'
          })
        }
      }).catch(function (error) {
        console.log(error)
      })
    },
    getCommodityInfo(commNo) {
      this.$router.push({
        path: '/CommodityInfo', query: {
          commNo: commNo
        }
      })
    },
    // 切换左侧菜单
    toggleMenuItem(data) {
      this.menuTab.forEach(item => {
        item.isActive = false
      })
      data.isActive = true
    },
    // element-ui 上传图片的方法 头像的！！！！
    handleProfilePhotoSuccess() {
      this.$notify({
        title: '成功',
        message: '上传成功！',
        type: 'success'
      })
    },
    // 上传前 头像的！！！！
    beforeProfilePhotoUpload(file) {
      const isJPG = file.type === 'image/jpeg' || file.type === 'image/jpg' || file.type === 'image/png'
      const isLt10M = file.size / 1024 / 1024 < 5

      if (!isJPG) {
        this.$message.error('上传的头像图片只能是 JPG/PNG/JPEG 格式!')
        return false
      }
      if (!isLt10M) {
        this.$message.error('上传的头像图片大小不能超过 10MB!')
        return false
      }
      // console.log(file)
      // 创建临时的路径来展示图片
      var windowURL = window.URL || window.webkitURL
      this.userBean.profileUrl = windowURL.createObjectURL(file)
      return true
    },
    // element-ui 上传图片的方法 商品的！！！！！
    handleAvatarSuccess(res, file) {
      this.imageUrl = URL.createObjectURL(file.raw)
      this.imageFile = file
    },
    // 上传前 商品的！！！！！
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg' || file.type === 'image/jpg' || file.type === 'image/png'
      const isLt2M = file.size / 1024 / 1024 < 5

      if (!isJPG) {
        this.$message.error('上传商品图片只能是 JPG/PNG/JPEG 格式!')
        return false
      }
      if (!isLt2M) {
        this.$message.error('上传商品图片大小不能超过 10MB!')
        return false
      }
      // 创建临时的路径来展示图片
      var windowURL = window.URL || window.webkitURL
      this.imageUrl = windowURL.createObjectURL(file)
      // 重新写一个表单上传的方法
      this.imageFile.append('commPicList', file, file.name)
      console.log(this.imageFile)
      return false
    },
    //商品标签转换
    select(commType) {
      console.log(commType)
      if (commType === '衣物') {
        this.commodity.commTag = 0
      } else if (commType === '数码') {
        this.commodity.commTag = 1
      } else if (commType === '食品') {
        this.commodity.commTag = 2
      } else if (commType === '图书') {
        this.commodity.commTag = 3
      } else if (commType === '化妆品') {
        this.commodity.commTag = 4
      } else if (commType === '文具') {
        this.commodity.commTag = 5
      } else {
        this.commodity.commTag = 6
      }
      console.log(this.commodity)
    },
    publish() {
      if (this.userBean.authentication !== 2) {
        this.$notify({
          title: '错误',
          message: '尚未认证卖家身份',
          type: 'error'
        })
      }
      var param = this.imageFile
      param.append('commDesc', this.commodity.description)
      param.append('commName', this.commodity.name)
      param.append('commPrice', this.commodity.price)
      param.append('commStock', this.commodity.quantity)
      param.append('commTag', this.commodity.commTag)
      param.append('customTags', this.commodity.dynamicTags)
      param.append('isFreeShipping ', 1)
      Server.releaseComm(param, this.token).then(resp => {
        this.imageFile = ''
        if (resp.code === 1) {
          this.$notify({
            title: '成功',
            message: '创建商品成功，请等待审核',
            type: 'success'
          })
          this.initData()
          this.commodity = {
            name: '',
            description: '',
            quantity: 1,
            price: '',
            commTag: '',
            dynamicTags: ['包邮'],
            inputVisible: false,
            inputValue: ''
          }
          this.imageFile = new FormData()
          this.imageUrl = ''
          this.commType = ''
          this.menuTab[3].isActive = false
          this.menuTab[2].isActive = true
        } else {
          this.$notify({
            title: '失败',
            message: '商品上传失败',
            type: 'error'
          })
          this.commodity = {
            name: '',
            description: '',
            quantity: 1,
            price: '',
            commTag: '',
            dynamicTags: ['包邮'],
            inputVisible: false,
            inputValue: ''
          }
          this.imageFile = new FormData()
          this.imageUrl = ''
          this.commType = ''
        }
      }).catch(function (error) {
        this.$notify.error({
          title: '错误',
          message: '发布失败'
        })
        console.log(error)
      })
    },
    //自定义标签
    handleClose(tag) {
      this.commodity.dynamicTags.splice(this.commodity.dynamicTags.indexOf(tag), 1);
    },
    showInput() {
      this.commodity.inputVisible = true;
      // eslint-disable-next-line no-unused-vars
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    handleInputConfirm() {
      let inputValue = this.commodity.inputValue;
      if (inputValue) {
        this.commodity.dynamicTags.push(inputValue);
      }
      this.commodity.inputVisible = false;
      this.commodity.inputValue = '';
    },
    //  数据格式化
    rTime: function (date) {
      var date1 = new Date(date).toJSON();
      return new Date(+new Date(date1) + 8 * 3600 * 1000).toISOString().replace(/T/g, ' ').replace(/\.[\d]{3}Z/, '')
    },
  },
  computed: {}
}
</script>
<style lang="scss">
#user {

  .demo-table-expand {
    font-size: 0;
  }

  .demo-table-expand label {
    width: 90px;
    color: #99a9bf;
  }

  .demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
  }

  .header {
  }

  .nav {
    background: #e2231a;
    height: 80px;
  }

  .content {
    background: #f5f5f5;

    .container {
      display: flex;
      width: 1210px;
      margin: 80px auto;
      padding: 0 4rem 2rem 0;
      min-height: 1000px;
      background: #fff;
      border-radius: 4px;
      box-shadow: 10px 5px 30px #99a9bf;

      .ct-left {
        height: 92%;
        margin: 40px 70px;
        border-right: 1px solid #dddddd;

        .img-wrap {
          width: 120px;
          height: 120px;
          padding: 5px;
          border: 1px solid #EAEAEA;

          img {
            width: 100%;
            height: 100%;
          }

          .avatar-uploader {
            height: 100%;
          }

          .el-upload {
            width: 100%;
            height: 100%;
            line-height: 120px;
            position: relative;

            .img-bottom {
              position: absolute;
              top: 0;
              left: 0;
              right: 0;
              bottom: 0;
              text-align: center;
              line-height: 130px;
              background-color: rgba($color: #000000, $alpha: .2);
              font-size: 14px;
              color: #fff;
              display: none;
            }
          }

          .el-upload:hover .img-bottom {
            display: block;
          }
        }

        .menu-tab-item {
          height: 70px;
          width: 125px;
          text-align: right;
          line-height: 70px;
          font-size: 15px;
          cursor: pointer;
        }

        .isActive {
          color: #409efe;
        }

        .menu-tab-item:hover {
          color: #409efe;
        }

        .menu-tab-item:focus {
          color: #409efe;
          outline: none;
        }
      }

      .ct-right {
        height: 92%;
        margin-top: 40px;

        .right-item {
          display: flex;
          font-size: 15px;
          height: 50px;

          .item-label {
            width: 50px;
          }
        }

        .publish {
          .right-item {
            line-height: 50px;
            margin-bottom: 20px;
          }

          .el-tag + .el-tag {
            margin-left: 10px;
          }

          .button-new-tag {
            margin-left: 10px;
            height: 32px;
            line-height: 30px;
            padding-top: 0;
            padding-bottom: 0;
          }

          .input-new-tag {
            width: 90px;
            margin-left: 10px;
            vertical-align: bottom;
          }
        }

        .avatar-uploader .el-upload {
          border: 1px dashed #d9d9d9;
          border-radius: 6px;
          cursor: pointer;
          position: relative;
          overflow: hidden;
        }

        .avatar-uploader .el-upload:hover {
          border-color: #409EFF;
        }

        .avatar-uploader-icon {
          font-size: 28px;
          color: #8c939d;
          width: 178px;
          height: 178px;
          line-height: 178px;
          text-align: center;
        }

        .avatar {
          width: 178px;
          height: 178px;
          display: block;
        }
      }
    }
  }
}
</style>
