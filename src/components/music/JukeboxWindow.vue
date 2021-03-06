<template
  ><!-- 150, 209 -->
  <window-frame
    titleText="再生中BGM一覧"
    fixSize="222, 274"
    display-property="private.display.jukeboxWindow"
    align="right-bottom"
    :isBanClose="true"
    @add="add"
    @remove="remove"
  >
    <div class="contents">
      <master-volume-component />
      <template v-for="bgmObj in playList">
        <b-g-m-youtube-component
          v-if="/www\.youtube\.com/.test(bgmObj.url)"
          :bgm="bgmObj"
          :key="bgmObj.key"
          :bgmKey="bgmObj.key"
          :ref="bgmObj.key"
          :tag="bgmObj.tag || ''"
          :isLoop="bgmObj.isLoop || false"
          :title="bgmObj.title || ''"
          :initVolume="bgmObj.volume || 0.5"
          :url="bgmObj.url || ''"
          @end="remove(bgmObj.key)"
          :startSecond="bgmObj.start || 0"
          :endSecond="bgmObj.end || 0"
          :fadeIn="bgmObj.fadeIn || 0"
          :fadeOut="bgmObj.fadeOut || 0"
        />
        <b-g-m-file-component
          v-if="!/www\.youtube\.com/.test(bgmObj.url)"
          :bgm="bgmObj"
          :key="bgmObj.key"
          :bgmKey="bgmObj.key"
          :ref="bgmObj.key"
          :tag="bgmObj.tag || ''"
          :isLoop="bgmObj.isLoop || false"
          :title="bgmObj.title || ''"
          :initVolume="bgmObj.volume || 0.5"
          :creditUrl="bgmObj.creditUrl"
          :url="bgmObj.url || ''"
          @end="remove(bgmObj.key)"
          :startSecond="bgmObj.start || 0"
          :endSecond="bgmObj.end || 0"
          :fadeIn="bgmObj.fadeIn || 0"
          :fadeOut="bgmObj.fadeOut || 0"
        />
      </template>
    </div>
  </window-frame>
</template>

<script lang="ts">
import WindowFrame from "../WindowFrame.vue";
import WindowMixin from "../WindowMixin.vue";

import BGMYoutubeComponent from "@/components/music/component/BGMYoutubeComponent.vue";
import MasterVolumeComponent from "@/components/music/component/MasterVolumeComponent.vue";
import BGMFileComponent from "@/components/music/component/BGMFileComponent.vue";

import { Action, Getter } from "vuex-class";
import { Component, Mixins } from "vue-mixin-decorator";
import { Watch } from "vue-property-decorator";

@Component({
  components: {
    WindowFrame,
    BGMFileComponent,
    BGMYoutubeComponent,
    MasterVolumeComponent
  }
})
export default class JukeboxWindow extends Mixins<WindowMixin>(WindowMixin) {
  @Action("windowClose") private windowClose: any;
  @Action("windowOpen") private windowOpen: any;
  @Getter("bgmList") private bgmList: any;

  private playList: any[] = [];

  private add(addBgmObj: any): void {
    if (!addBgmObj) return;

    // 強制リスタートじゃなければ、もともと再生されてる曲を中断しない。
    if (!addBgmObj.isReject && !addBgmObj.forceReset) {
      const bgmObj: any = this.playList.filter(
        (bgmObj: any) => bgmObj.key === addBgmObj.key
      )[0];
      if (bgmObj) {
        return;
      }
    }

    // タグが同じものはプレイリストから削除する
    const delList = this.playList.filter(plObj => {
      const bgmObj = this.bgmList.filter(
        (bgmObj: any) => bgmObj.key === plObj.key
      )[0];
      if (!bgmObj) {
        // 見つからなかったらタイミング悪く削除されたということなので、削除リストに追加
        return true;
      }
      return addBgmObj.tag === bgmObj.tag;
    });
    delList.forEach(delObj => {
      const index = this.playList.indexOf(delObj);
      this.playList.splice(index, 1);
    });

    // 追加処理
    if (addBgmObj.url !== "") {
      setTimeout(() => {
        this.playList.unshift(addBgmObj);
        this.windowOpen("private.display.jukeboxWindow");
      }, 0);
    } else {
      if (this.playList.length === 0) {
        this.windowClose("private.display.jukeboxWindow");
      }
    }
  }

  private remove(bgmKey: string): void {
    const index = this.playList.findIndex(
      (bgmObj: any) => bgmObj.key === bgmKey
    );
    this.playList.splice(index, 1);

    if (this.playList.length === 0) {
      this.windowClose("private.display.jukeboxWindow");
    }
  }
}
</script>

<style scoped lang="scss">
@import "../common.scss";

.contents {
  @include flex-box(column);
  position: absolute;
  height: 100%;
  width: 100%;
  font-size: 12px;
  overflow-y: auto;
}

.bgmComponent {
  padding-top: 5px;
  margin-top: 5px;
  border-top: 2px solid gray;
}
</style>
