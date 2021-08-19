<template>
  <div class="content">
    <div class="row" v-for="(item, index) in showTextArr" :key="index">
      <span class="rowIndex">{{ index + 1 }}</span>
      <span class="rowText" v-html="item"></span>
    </div>
  </div>
</template>

<script>
import textData from './text';
import diff_match_patch from './diff_match_patch_uncompressed';

const oldText = textData.text1;
const newText = textData.text2;
const dmp = new diff_match_patch();

/**
 * @Author: 王林25
 * @Date: 2021-08-17 20:17:06
 * @Desc: 根据dp数组收集公共子序列
 */
let collect = (dp, text1, text2, i, j, arr1, arr2) => {
    if (i === 0 || j === 0) {
        return;
    }
    if (text1[i - 1] === text2[j - 1]) {
        arr1.push(i - 1);
        arr2.push(j - 1);
        return collect(dp, text1, text2, i - 1, j - 1, arr1, arr2);
    } else {
        if (dp[i][j - 1] > dp[i - 1][j]) {
            return collect(dp, text1, text2, i, j - 1, arr1, arr2);
        } else {
            return collect(dp, text1, text2, i - 1, j, arr1, arr2);
        }
    }
};

/**
 * @Author: 王林25
 * @Date: 2021-08-17 20:16:47
 * @Desc: 求最长公共子序列
 */
let longestCommonSubsequence = (text1, text2) => {
    let arr1 = [];
    let arr2 = [];
    let m = text1.length;
    let n = text2.length;
    // 初始化二维数组
    let dp = new Array(m + 1).fill(0);
    dp.forEach((item, index) => {
        dp[index] = new Array(n + 1).fill(0);
    });
    for (let i = 1; i <= m; i++) {
    // 因为i和j都是从1开始的，所以要减1
        let t1 = text1[i - 1];
        for (let j = 1; j <= n; j++) {
            let t2 = text2[j - 1];
            // 情况1
            if (t1 === t2) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                // 情况2
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    collect(dp, text1, text2, m, n, arr1, arr2);
    arr1.sort((a, b) => {
        return a - b;
    });
    arr2.sort((a, b) => {
        return a - b;
    });
    return [arr1, arr2];
};

/**
 * @Author: 王林25
 * @Date: 2021-08-17 20:24:06
 * @Desc: 获取删除和新增的索引
 */
let getDiffList = (text1, text2, arr1, arr2) => {
    let delList = [];
    let addList = [];
    // 遍历旧的字符串
    for (let i = 0; i < text1.length; i++) {
    // 旧字符串里不在公共子序列里的位置的字符代表是被删除的
        if (!arr1.includes(i)) {
            delList.push(i);
        }
    }
    // 遍历新字符串
    for (let i = 0; i < text2.length; i++) {
    // 新字符串里不在公共子序列里的位置的字符代表是新增的
        if (!arr2.includes(i)) {
            addList.push(i);
        }
    }
    return {
        delList,
        addList
    };
};

export default {
    data () {
        return {
            oldTextArr: [],
            newTextArr: [],
            showTextArr: []
        };
    },
    mounted () {
        // this.diff();
        this.diffAll();
    },
    methods: {
        /**
         * @Author: 王林25
         * @Date: 2021-08-18 19:58:04
         * @Desc: 直接diff整个文本
         */
        diffAll () {
            let diffList = dmp.diff_main(oldText, newText);
            let htmlStr = '';
            diffList.forEach((item) => {
                switch (item[0]) {
                case 0:
                    htmlStr += item[1];
                    break;
                case 1:
                    htmlStr += `<span class="add">${item[1]}</span>`;
                    break;
                case -1:
                    htmlStr += `<span class="del">${item[1]}</span>`;
                    break;
                default:
                    break;
                }
            });
            this.showTextArr = htmlStr.split(/\n+/);
        },

        /**
     * @Author: 王林25
     * @Date: 2021-08-18 09:37:38
     * @Desc: 对比每一行文本
     */
        diff () {
            // 新旧文本分割成数组
            this.oldTextArr = oldText.split(/\n+/g);
            this.newTextArr = newText.split(/\n+/g);
            // 如果新旧行数不一样，用空字符串来补齐
            let oldTextArrLen = this.oldTextArr.length;
            let newTextArrLen = this.newTextArr.length;
            let diffRow = Math.abs(oldTextArrLen - newTextArrLen);
            if (diffRow > 0) {
                let fixArr = oldTextArrLen > newTextArrLen ? this.newTextArr : this.oldTextArr;
                for (let i = 0; i < diffRow; i++) {
                    fixArr.push('');
                }
            }
            let len = this.newTextArr.length;
            for (let row = 0; row < len; row++) {
                // 如果新旧文本完全相同就不用比较了
                if (this.oldTextArr[row] === this.newTextArr[row]) {
                    this.showTextArr.push(this.newTextArr[row]);
                    continue;
                }
                // 否则计算新旧文本的最长公共子序列的位置
                let [arr1, arr2] = longestCommonSubsequence(
                    this.oldTextArr[row],
                    this.newTextArr[row]
                );
                // 进行标注操作
                this.mark(row, arr1, arr2);
            }
        },

        /**
     * @Author: 王林25
     * @Date: 2021-08-18 09:49:09
     * @Desc: 标注删除和新增
     */
        mark (row, oldArr, newArr) {
            let oldText = this.oldTextArr[row];
            let newText = this.newTextArr[row];
            // 获取删除和新增的位置索引
            let { delList, addList } = getDiffList(oldText, newText, oldArr, newArr);
            let addTagLength = 0;
            // 遍历新增位置数组
            addList.forEach((index) => {
                let pos = index + addTagLength;
                // 截取当前位置前面的字符串
                let pre = newText.slice(0, pos);
                // 截取后面的字符串
                let post = newText.slice(pos + 1);
                newText = pre + `<span class="add">${newText[pos]}</span>` + post;
                addTagLength += 25;// <span class="add"></span>的长度为25
            });

            // 插入的字符所占的位置
            let insertStrLength = 0;
            // 遍历删除的索引数组
            delList.forEach((index) => {
                let newIndex = this.getDelIndexInNewTextIndex(index, oldArr, newArr);
                // 前面新增的字符数量
                let addLength = addList.filter((item) => {
                    return item < newIndex;
                }).length;
                // 前面没有变化的字符数量
                let noChangeLength = newArr.filter((item) => {
                    return item < newIndex;
                }).length;
                // 加上新插入字符所占的总长度
                let pos = insertStrLength + addLength * 26 + noChangeLength;
                let pre = newText.slice(0, pos);
                let post = newText.slice(pos);
                newText = pre + `<span class="del">${oldText[index]}</span>` + post;
                // <span class="del">x</span>的长度为26
                insertStrLength += 26;
            });

            this.showTextArr.push(newText);
        },

        /**
         * @Author: 王林25
         * @Date: 2021-08-18 18:43:00
         * @Desc: 获取被删除字符在新文本里的索引
         */
        getDelIndexInNewTextIndex (index, oldArr, newArr) {
            for (let i = oldArr.length - 1; i >= 0; i--) {
                if (index > oldArr[i]) {
                    return newArr[i] + 1;
                }
            }
            return 0;
        }
    }
};
</script>

<style scoped lang="less">
.content {
  padding: 10px 0;
  overflow-x: hidden;

  .row {
    display: flex;

    /deep/ .add {
      background-color: #ddfbe6;
    }

    /deep/ .del {
      background-color: #f9d7dc;
      text-decoration: line-through;
    }

    .rowIndex {
      width: 30px;
      flex-shrink: 0;
      background-color: #f5f5f5;
      padding: 5px;
      display: flex;
      justify-content: flex-end;
      align-items: center;
    }

    .rowText {
      padding: 0 10px;
      color: rgba(0, 0, 0, 0.7);
      word-break: break-all;
      line-height: 30px;
    }
  }
}
</style>
