<template>
  <n-config-provider :theme="theme">
    <n-layout embedded content-style="padding: 54px;height:100vh">
      <n-card>
        <div style="height: 80vh; position: relative">
          <n-layout position="absolute">
            <n-layout-header style="height: 64px; padding: 24px;    font-size: 20px;
      font-weight: 700;" bordered>
              EdgeGPT Next Web <n-button @click="theme = darkTheme" style="position: fixed;
    right: 0;
    margin-right: 10%;">
                深色
              </n-button>
              <n-button @click="theme = null" style="position: fixed;
    right: 0;
    margin-right: 5%;">
                浅色
              </n-button>
            </n-layout-header>
            <n-layout has-sider position="absolute" style="top: 64px; bottom: 64px">
              <n-layout-sider bordered content-style="padding: 24px;" show-trigger collapse-mode="width"
                :collapsed-width="128" :width="240" :native-scrollbar="false" :inverted="inverted">
                <n-menu :inverted="inverted" :collapsed-width="64" :collapsed-icon-size="22" :options="menuOptions"
                  @update:value="handleUpdateValue" />
              </n-layout-sider>
              <n-layout content-style="padding: 24px;">
                <n-list>
                  <n-list-item v-for="item in data" :key="item.id">
                    <n-space>
                      <n-avatar round size="medium" :src="item.avatar" />
                      {{ item.name }}
                    </n-space>
                    <MdEditor previewOnly v-model="item.content"></MdEditor>
                    {{ item.time }}
                  </n-list-item>
                </n-list><n-input
      v-model:value="value"
      type="textarea"
      placeholder="你的消息,Ctrl+Enter发送"
      @keydown.enter="keyDown"
    />
    <n-button type="success" @click="sendMessage">
      发送
    </n-button>

                
              </n-layout>
            </n-layout>
            <n-layout-footer bordered position="absolute" style="height: 64px; padding: 24px">
              城府路
            </n-layout-footer>
          </n-layout>
        </div>
      </n-card>
    </n-layout>
  </n-config-provider>
</template>
<script lang="ts">
import { h, defineComponent, ref, Component } from 'vue'
import { NIcon } from 'naive-ui'
import {
  PersonOutline as PersonIcon,
} from '@vicons/ionicons5'
let active = ref("");
import MdEditor from 'md-editor-v3';
import 'md-editor-v3/lib/style.css';
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';
dayjs.extend(relativeTime);
import { useRouter } from 'vue-router'
import { w3cwebsocket as W3CWebSocket } from "websocket";
const ws = ref(null)
import { darkTheme } from 'naive-ui'
import type { GlobalTheme } from 'naive-ui'
function renderIcon(icon: Component) {
  return () => h(NIcon, null, { default: () => h(icon) })
}

const menuOptions = [
  {
    label: '新的聊天',
    key: '/',
    icon: renderIcon(PersonIcon)
  },
]

export default defineComponent({
  components: {
    MdEditor
  },
  mounted(){
    let location = window.location.host
    let protocol = ""
    if(window.location.protocol == "https:"){
      protocol = "wss://"
    }else{
      protocol = "ws://"
    }
    ws.value = new W3CWebSocket(`${protocol}${location}/api/ws`);
    ws.value.onopen = function() {
        console.log('WebSocket Client Connected');
      };

      ws.value.onmessage = function(message) {
        console.log('Received: ' + message.data);
        message.value = message.data
      };

      ws.value.onerror = function() {
        console.log('Connection Error');
      };

      ws.value.onclose = function() {
        console.log('echo-protocol Client Closed');
      };
  },
  methods: {
    keyDown(e: any){
      if(e.ctrlKey && e.keyCode==13) {   //用户点击了ctrl+enter触发
        this.sendMessage()
	      this.value = '';
	    }
    },
    sendMessage(){
      return this.value
    },
  },
  setup() {
    const router = useRouter()
    const handleUpdateValue = (key: string, item: any) => {
      active.value = key;
      router.push(key);
    };
    /*const data = [
      { id: 1, name: 'Alice', avatar: 'https://cdn.staticfile.org/emoji-datasource-apple/14.0.0/img/apple/64/1f603.png' ,time:dayjs().fromNow(),content: "helloworld"},
      { id: 2, name: 'Bob', avatar: 'https://cdn.staticfile.org/emoji-datasource-apple/14.0.0/img/apple/64/1f603.png' ,time:dayjs().fromNow(),content: "helloworld"},
      { id: 3, name: 'Charlie', avatar: 'https://cdn.staticfile.org/emoji-datasource-apple/14.0.0/img/apple/64/1f603.png' ,time:dayjs().fromNow(),content: "helloworld"}
    ]*/
    const data:any = []
    return {
      inverted: ref(false),
      menuOptions,
      darkTheme,
      theme: ref<GlobalTheme | null>(null),
      handleUpdateValue,
      data,
      value: ref(""),
    }
  }
})
</script>
<style>
.n-list-item__divider {
  display: none;
}
</style>