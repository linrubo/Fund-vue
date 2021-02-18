<template>
    <div class="fund">
        <header>
            <p>总金额：{{ totalMoney }}</p>
            <p>总份额：{{ totalAmount.toFixed(2) }}</p>
            <p>平摊成本：{{ mean }}</p>
            <p>收益：{{ profit }}</p>
            <p>收益率：{{ profitability.toFixed(2) }}%</p>
            <p>最大浮亏：{{ loss.toFixed(2) }}%</p>
            <p>{{ date }} 买入 {{ money }}</p>
        </header>
        <main class="box">
            <ul>
                <li v-for="(item, index) in history"
                    :key="index"
                    :class="{blank: item.money === 0}"
                    :style="{
                        width: `${item.width / radix}px`,
                        height: `${item.price * radix}px`,
                        background: item.background,
                    }"
                >
                    {{ item.date }} | {{ item.money }} | {{ item.amount }} | {{ item.price}}
                </li>
            </ul>
            <div class="mean" :style="{height: `${mean * radix}px`}"></div>
            <div class="last" :style="{height: `${last * radix}px`}"></div>
        </main>
        <footer>
            <select v-model="strategy" :disabled="mode !== ''">
                <option value="va">价值平均策略</option>
                <option value="dca">成本平均策略</option>
            </select>
            <button :disabled="mode === 'auto'" @click="manual">手动</button>
            <button :disabled="mode === 'manual'" @click="auto">自动</button>
            <button :disabled="paused" @click="pause">暂定</button>
            <button @click="reset">重置</button>
        </footer>
    </div>
</template>

<script>
import transactionVA from '../datas/transaction-va.js';
import transactionDCA from '../datas/transaction-dca.js';

export default {
    name: "Fund",
    data() {
        return {
            // 收缩比例
            radix: 100,
            // 策略名称
            strategy: 'va',
            // 总投入金额
            totalMoney: 0,
            // 累计份额
            totalAmount: 0,
            // 平均成本
            mean: 0,
            // 最新净值
            last: 0,
            // 收益
            profit: 0,
            // 收益率
            profitability: 0,
            // 最大浮亏
            loss: 0,
            // 当前日期
            date: '0000-00-00',
            // 当天买入金额
            money: 0,
            // 交易记录
            history: [],
            // 交易记录索引值
            index: 0,
            // 计时器
            timer: null,
            // 计时器间隔实践
            interval: 100,
            // 执行方式，auto | manual
            mode: '',
            // 是否禁用暂停按钮
            paused: true,
        }
    },
    methods: {
        update(item) {
            this.totalMoney += item.money;
            this.totalAmount += item.amount;
            this.mean = (this.totalMoney / this.totalAmount).toFixed(4);
            this.last = item.price;
            this.profit = (this.totalAmount * item.price - this.totalMoney).toFixed(2);
            this.profitability = (this.profit / this.totalMoney * 100);
            this.date = item.date;
            this.money = item.money;

            if (this.profitability < this.loss) {
                this.loss = this.profitability;
            }
        },
        buy() {
            let item;

            if (this.strategy === 'va') {
                item = transactionVA[this.index];
            }
            if (this.strategy === 'dca') {
                item = transactionDCA[this.index];
            }

            if (item) {
                item.width = item.amount || this.radix * 5;
                if (item.money == 0) {
                    item.background = 'rgba(0, 0, 0, .1)';
                } else {
                    item.background = `#${Math.random().toString(16).slice(-6)}`;
                }

                this.history.push(item);
                this.update(item);

                this.index += 1;
            } else {
                this.pause();
            }
        },
        manual() {
            this.mode = 'manual';
            this.buy();
        },
        auto() {
            this.mode = 'auto';
            this.paused = false;
            this.timer = setInterval(() => {
                this.buy()
            }, this.interval);
        },
        pause() {
            this.paused = true;
            clearInterval(this.timer);
        },
        reset() {
            this.pause();

            this.strategy = 'va';
            this.totalMoney = 0;
            this.totalAmount = 0;
            this.mean = 0;
            this.last = 0;
            this.profit = 0;
            this.profitability = 0;
            this.loss = 0;
            this.date = '0000-00-00';
            this.money = 0;
            this.history = [];
            this.index = 0;
            this.mode = '';
            this.paused = true;
        }
    }
}
</script>

<style>
main {
    position: relative;
    height: 300px;
    margin: 20px 0;
    border: 1px solid #000;
    overflow: auto;
}
main ul {
    list-style: none;
    margin: 0;
    padding: 0;
    height: 100%;
    display: flex;
    align-items: flex-end;
}
main ul li {
    /* flex-shrink: 0; */
    text-indent: -999em;
}
main ul li:nth-child(even) {
    opacity: 0.8;
}
main ul li.blank {
    border-top: 1px solid red;
}
main .mean,
main .last {
    position: absolute;
    right: 0;
    bottom: 0;
    left: 0;
    overflow: hidden;
}
main .mean {
    border-top: 1px dotted blue;
}
main .last {
    border-top: 1px dotted red;
}
footer {
    display: flex;
    justify-content: center;
    align-items: center;
}
footer > * {
    margin: 0 1em;
}
</style>