<template>
  <div class="follow">
    <div class="follow-nav">
      <div class="follow-nav-header">
        <div class="follow-nav-header-label">我的关注</div>
        <div class="follow-nav-header-action">
          <span @click="showCreateListModal"><i class="fas fa-plus-circle follow-nav-header-action-add"></i></span>
        </div>
      </div>

      <ul class="follow-list">
        <li class="follow-list-item">
          <router-link class="follow-list-item-link" to="/list/-1">
            <i class="follow-list-item-link-icon fas fa-globe"></i>
            <span class="follow-list-item-link-name">全部关注</span>
          </router-link>
        </li>
        <li v-for="list in followLists" :key="list.id" class="follow-list-item">
          <router-link :to="{ path: `/list/${list.id}` }" class="follow-list-item-link">
            <i class="follow-list-item-link-icon fas fa-users"></i>
            <span class="follow-list-item-link-name">{{ list.name }}</span>
            <span class="follow-list-item-link-count">{{ list.mids.length }}</span>
            <ul class="follow-list-item-link-dropdown dropdown-menu">
              <li class="dropdown-menu-item dropdown-menu-item-rename" @click="showRenameListModal(list.id,list.name)">修改名称</li>
              <li class="dropdown-menu-item dropdown-menu-item-delete" @click="handleDeleteList(list.id)">删除</li>
            </ul>
          </router-link>
        </li>
      </ul>
    </div>

    <!-- modal-->
    <div v-show="isCreateListModalVisible" id="modal-create-list" class="modal">
      <div class="modal-content">
        <div class="modal-header">
          <h3 class="modal-title">新建分组</h3>
          <span class="modal-close" @click="handleCreateListModalCancel">×</span>
        </div>
        <div class="modal-body">
          <input class="modal-input" v-model="createListModalValue"/>
        </div>
        <div class="modal-footer">
          <button class="modal-button modal-button-ok" @click="handleCreateListModalSuccess">确定</button>
          <button class="modal-button modal-button-cancel" @click="handleCreateListModalCancel">取消</button>
        </div>
      </div>
    </div>

    <div v-show="isRenameListModalVisible" id="modal-rename-list" class="modal">
      <div class="modal-content">
        <div class="modal-header">
          <h3 class="modal-title">重命名分组</h3>
          <span class="modal-close" @click="handleRenameListModalCancel">×</span>
        </div>
        <div class="modal-body">
          <label>
            <input class="modal-input" v-model="renameListName"/>
          </label>
        </div>
        <div class="modal-footer">
          <button class="modal-button modal-button-ok" @click="handleRenameListModalSuccess">确定</button>
          <button class="modal-button modal-button-cancel" @click="handleRenameListModalCancel">取消</button>
        </div>
      </div>
    </div>

    <div class="follow-content">
      <router-view></router-view>
    </div>
  </div>
</template>

<script>
import { FollowListService } from '@/app/services'

export default {
  name: 'Follow',
  data () {
    return {
      mouseOverListId: -1, // 我的关注 = 全部关注
      followLists: [],
      createListModalValue: '', // 创建分组的对话框的文本值
      isCreateListModalVisible: false, // 创建分组的显示状态
      isCreateListModalSuccessLoading: false, // 创建分组成功后的通知信息
      renameListName: '', // 重命名列表的名称
      renameListId: 0, // 重命名列表的id
      isRenameListModalVisible: false, // 重命名分组的显示状态
      isRenameListModalSuccessLoading: false // 重命名分组成功后的通知
    }
  },
  created () {
    this.initService()
    this.loadData()
  },
  methods: {
    initService () {
      this.followListService = new FollowListService()
    },
    loadData () {
      this.followListService.getFollowLists().subscribe((followedLists) => {
        this.followLists = followedLists
      })
    },
    mouseEnter (listId) {
      this.mouseOverListId = listId
    },
    mouseleave () {
      setTimeout(() => {
        this.mouseOverListId = -1
      }, 1000)
    },
    handleDeleteList (id) {
      // get all follow list ids, then check whether include id that parameter passes
      if (this.followLists.map((followList) => followList.id).includes(id)) {
        this.followListService.deleteFollowList(id).subscribe((followLists) => {
          this.followLists = followLists
          console.log('分组删除成功')
          // todo reset active router
          // this.router.navigateByUrl("follow/list/-1");
        })
      }
    },
    // show create list modal
    showCreateListModal () {
      this.createListModalValue = ''
      this.isCreateListModalVisible = true
    },

    // reset modal state
    handleCreateListModalCancel () {
      this.createListModalValue = ''
      this.isCreateListModalVisible = false
    },

    handleCreateListModalSuccess () {
      // use fast failed and exit early methodology
      if (this.followLists.length >= 10) {
        console.log('最多只能有十个分组')
        return
      }

      if (this.isEmpty(this.createListModalValue)) {
        console.log('分组名字不能为空')
        return
      }

      if (this.createListModalValue.length > 20) {
        console.log('分组名字过长，最多20个字符')
        return
      }

      if (this.followLists.map((followList) => followList.name).includes(this.createListModalValue)) {
        console.log('分组名字重复')
        return
      }

      this.isCreateListModalSuccessLoading = true
      this.followListService.addFollowList(this.createListModalValue).subscribe((followLists) => {
        this.followLists = followLists
        console.table(followLists)
        // cancel loading
        this.isCreateListModalSuccessLoading = false
        // hidden modal
        this.isCreateListModalVisible = false
        console.log('分组创建成功')
      })
    },
    /**
     * "", "    ", => true
     *
     * @param str
     * @return {boolean|*|string}
     */
    isEmpty (str) {
      return !str || str.trim().length === 0
    },
    showRenameListModal (id, name) {
      this.renameListId = id
      this.renameListName = name
      this.isRenameListModalVisible = true
    },

    handleRenameListModalCancel () {
      this.isRenameListModalVisible = false
    },

    handleRenameListModalSuccess () {
      if (this.renameListName.length <= 10) {
        if (!this.followLists.map((followList) => followList.name).includes(this.renameListName)) {
          this.isRenameListModalSuccessLoading = true
          this.followListService.renameFollowList(this.renameListId, this.renameListName).subscribe((followLists) => {
            console.log(followLists)
            this.followLists = followLists
            this.isRenameListModalSuccessLoading = false
            this.isRenameListModalVisible = false
            console.log('分组名字修改成功')
          })
        } else {
          console.log('分组名字重复')
        }
      } else {
        console.log('分组名字过长')
      }
    },
    foo () {
      console.log('foo')
    }
  }
}
</script>

<style lang="scss" scoped>
.follow {
  color: #666262;
  display: flex;
  flex-direction: row;

  &-nav {
    flex: 0 0 12.5em;
    font-size: 0.85rem;
    border-right: #e2e2e2 solid 1px;

    &-header {
      display: flex;
      flex-direction: row;
      align-items: center;
      padding-left: 16px;

      &-label {
        flex: 1;
        padding: 16px 0;
      }

      &-action {
        justify-self: end;
        margin-right: 10px;

        &-add {
          width: 20px;
          height: 20px;
          color: #4cd495;
          cursor: pointer;
        }
      }
    }
  }

  &-list {
    &-item {
      list-style: none;
      margin-top: 4px;
      cursor: pointer;

      &-link {
        display: flex;
        flex-direction: row;
        align-items: center;
        text-decoration: none;
        color: #666262;
        padding: 12px 8px 12px 16px;
        position: relative;

        &:hover {
          color: #42b983;
        }

        &.router-link-exact-active {
          color: #42b983;
          background-color: rgba(66, 185, 131, 0.1);
        }

        &-icon {
          width: 20px;
          height: 20px;

          margin-right: 10px;
        }

        &-name {
          flex: 1;
          margin-right: 10px;
        }

        &-count {
          justify-self: end;
        }

        &-dropdown {
          border: #e2e2e2 solid 1px;
          border-radius: 5px;
          position: absolute;
          top: 100%;
          right: 0;
          list-style: none;
          z-index: 1;
        }
      }
    }
  }

  &-content {
    flex: 1;
  }
}

.modal {
  position: fixed;
  z-index: 9999;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, .2);

  display: flex;
  align-items: center;
  justify-content: center;

  &-content {
    border: #e2e2e2 solid 1px;
    border-radius: 5px;
    background-color: white;
    width: 40%;

    display: flex;
    flex-direction: column;

    font-size: 1rem;
    color: #666262;
  }

  &-header {
    border-bottom: #e2e2e2 solid 1px;
    display: flex;
    flex-direction: row;
    align-items: center;
    padding: 6px 12px;
  }

  &-title {
    flex: 1;
    font-weight: 400;
    font-size: 1em;
  }

  &-close {
    font-size: 2em;
    cursor: pointer;
  }

  &-body {
    padding: 16px;
    flex: 1;
    border-bottom: #e2e2e2 solid 1px;
  }

  &-input {
    width: 100%;
    border: 1px solid rgba(147, 128, 108, 0.25);
    border-radius: 4px;
    padding: 0.5em 0.75em;
  }

  &-footer {
    display: flex;
    flex-direction: row-reverse;
    font-weight: 400;
    padding: 8px 12px;
  }

  &-button {
    outline: none;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    justify-self: end;
    padding: 8px 16px;
    border-radius: 5px;
    border: #e2e2e2 solid 1px;
    cursor: pointer;
    margin-left: 10px;

    &-ok {
      order: -1;
      background-color: #3da2ff;
      color: white;
    }

    &-cancel {
      order: 1;
      color: #666262;
    }
  }

}
</style>