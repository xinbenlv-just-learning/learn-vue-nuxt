<!--
  Copyright 2019 Google LLC

  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<template>
  <div class="diff-card d-flex flex-column">
    <div class="d-flex justify-content-between">
      <h5>{{ $t('Label-OriginalWikitext') }}</h5>
      <h5>{{ $t('Label-ChangedWikitext') }}</h5>
    </div>
    <table class="diff-content">
      <thead class="diff-header">
        <tr>
          <th colspan="2" />
          <th colspan="2" />
        </tr>
      </thead>
      <!-- eslint-disable-next-line vue/no-v-html -->
      <tbody v-html="processedDiffContent" />
    </table>
  </div>
</template>

<script lang="ts">
import { wikiToDomain } from '@/shared/utility-shared';
export default {
  props: {
    diffContent: {
      type: String,
      default: '',
    },
    wikiRevId: {
      type: String,
      default: '',
    },
    diffMetadata: {
      type: Object,
      default: null,
    },
  },
  data() {
    return {
      processedDiffContent: '',
    };
  },
  beforeMount() {
    this.processDiffContent(this.diffContent);
  },
  beforeUpdate() {
    this.processDiffContent(this.diffContent);
  },
  methods: {
    processDiffContent() {
      if (!window.DOMParser || !this.diffMetadata) {
        this.processedDiffContent = this.diffContent;
        return;
      }

      let diffContent = this.diffContent;
      // https://regex101.com/r/QwzU8z/3
      diffContent = diffContent.replace(
        /\[\[([^\]|]*)(\|?.*?)\]\]/gm,
        function(match, p1, p2) {
          let parsedText = new DOMParser().parseFromString(p1, 'text/html');
          const cleanedUpP1 = parsedText.querySelector('body').textContent;
          let articleName = cleanedUpP1.split('#')[0];
          articleName =
            articleName.charAt(0).toUpperCase() + articleName.slice(1);
          let className = 'new';

          parsedText = undefined;
          if (this.diffMetadata.links[articleName]) {
            className = 'exists';
          } else if (this.diffMetadata.iwlinks[articleName]) {
            className = 'exists';
          }

          this.diffMetadata.images.forEach((entry) => {
            if (cleanedUpP1.includes(entry)) {
              className = '';
            }
          });

          const link = `http://${
            wikiToDomain[this.wikiRevId.split(':')[0]]
          }/wiki/${cleanedUpP1}`;
          return `[[<a href="${link}" target="_blank" class="${className}">${p1}</a>${p2}]]`;
        }.bind(this),
      );

      this.processedDiffContent = diffContent;
    },
  },
};
</script>

<style lang="scss" scoped>
@import 'bootstrap/scss/_functions.scss';
@import 'bootstrap/scss/_variables.scss';
@import 'bootstrap/scss/_mixins.scss';
.diff-header {
  position: sticky;
  top: 0;
  background: #ffffff;
}

.diff-card a.new {
  color: #ba0000;
}

.diff-card a.exists {
  color: #0645ad;
}

.diff-card {
  font-size: 0.75rem;
  overflow-x: hidden;
  flex: 1 1 200px;

    @include media-breakpoint-down(lg) {
      font-size: 0.6rem;
    }
    @include media-breakpoint-down(md) {
      font-size: 0.5rem;
    }
    /* If the screen size is 600px wide or less, set the font-size of <div> to 30px */
    @include media-breakpoint-down(sm) {
      font-size: 0.4rem;
    }
  }
</style>
