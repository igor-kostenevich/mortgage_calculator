<template>
  <div class="wrapper">
    <div class="content container">
      <div class="content__column">
        <h1>Калькулятор ипотеки</h1>
        <div class="content-inputs">
          <div class="input-slider-group">
            <div class="range-info">
              <div class="range-label">Стоимость недвижимости</div>
              <input
                class="range-input"
                type="text"
                v-model.number="totalCost"
              />
            </div>
            <el-slider
              v-model="totalCost"
              :max="3000000"
              :step="500"
              :show-tooltip="false"
              label="Стоимость недвижимости"
            ></el-slider>
          </div>
          <div class="input-slider-group">
            <div class="range-info">
              <div class="range-label">Первоначальный взнос</div>
              <input
                class="range-input"
                type="text"
                v-model.number="initialFee"
              />
            </div>
            <el-slider
              v-model.number="initialFee"
              :max="2000000"
              :step="500"
              :show-tooltip="false"
              label="Первоначальный взнос"
            ></el-slider>
          </div>
          <div class="input-slider-group">
            <div class="range-info">
              <div class="range-label">Срок кредита</div>
            </div>
            <el-slider
              v-model="creditTerm"
              :max="216"
              :min="12"
              :step="12"
              :show-tooltip="false"
              label="Срок кредита"
              :marks="dataMarks.marks"
              class="el-slider-marks"
            ></el-slider>
          </div>
        </div>
        <div class="content__bank-selection select-bank">
          <h2 class="select-bank__title">Выбрать банк</h2>
          <app-select-bank-item
            v-for="bank in selectBanks"
            :rate="bank.rate"
            :key="bank.name"
            :class="{ active: bank.isActive }"
            @active-bank="isActiveBank(bank)"
          >{{ bank.name }}</app-select-bank-item>
        </div>
      </div>
      <div class="content__column">
        <div class="show-items">
          <div class="show-item">
            <div class="show-item__label">Сумма кредита</div>
            <div class="show-item__result">{{ currency(totalAmountOfCredit) }}</div>
          </div>
          <div class="show-item">
            <div class="show-item__label">Ежемесячный платеж</div>
            <div class="show-item__result">{{ currency(totalMounthlyPayment) }}</div>
          </div>
          <div class="show-item">
            <div class="show-item__label">Рекомендуемый доход</div>
            <div class="show-item__result">{{ currency(totalRecommendedIncome) }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import AppSelectBankItem from './components/AppSelectBankItem'
import { ElSlider } from 'element-plus'
import { currency } from './utils/currency'

export default {
  data () {
    return {
      totalCost: 0,
      initialFee: 0,
      creditTerm: 12,
      percent: 7,
      totalAmountOfCredit: 0,
      totalMounthlyPayment: 0,
      totalRecommendedIncome: 0,
      dataMarks: {
        marks: {
          12: '12',
          24: '24',
          36: '36',
          48: '48',
          60: '60',
          72: '72',
          84: '84',
          96: '96',
          108: '108',
          120: '120',
          132: '132',
          144: '144',
          156: '156',
          168: '168',
          180: '180',
          192: '192',
          204: '204',
          216: '216'
        },
        arrMarksItems: [],
        arrMarksDistance: []
      },
      selectBanks: [
        { name: 'Приватбанк', rate: 7, isActive: true },
        { name: 'Альфа-банк', rate: 7.5, isActive: false },
        { name: 'Мегабанк', rate: 6.9, isActive: false },
        { name: 'ОТП банк', rate: 7.1, isActive: false },
        { name: 'Ощадбанк', rate: 7.3, isActive: false }
      ],
      currency
    }
  },

  methods: {
    isActiveBank (bank) {
      if (!bank.isActive) {
        const oldActiveBank = this.selectBanks.find((item) => item.isActive)
        oldActiveBank.isActive = false
        bank.isActive = true
        this.percent = bank.rate
      }
    }
  },

  mounted () {
    this.dataMarks.arrMarksItems.push(...document.querySelectorAll('.el-slider__marks-stop'))
    this.dataMarks.arrMarksItems.forEach((item) => {
      const getStyleMarksDistance = parseFloat(getComputedStyle(item).left)
      this.dataMarks.arrMarksDistance.push(getStyleMarksDistance)
    })
  },

  updated () {
    // active breakpoint logic for creditTerm input
    if (document.querySelector('.el-slider-marks')) {
      const btnSlider = document.querySelector('.el-slider-marks .el-slider__button-wrapper')
      const getStyleButtonDistance = parseFloat(getComputedStyle(btnSlider).left)

      const arrMarksDone = this.dataMarks.arrMarksItems.filter((item, idx) => {
        if (idx > this.dataMarks.arrMarksDistance.indexOf(getStyleButtonDistance)) {
          item.classList.remove('done')
        }
        return idx <= this.dataMarks.arrMarksDistance.indexOf(getStyleButtonDistance)
      })

      arrMarksDone.forEach((item) => {
        item.classList.add('done')
      })
    }

    // calculator logic
    this.totalAmountOfCredit = this.totalCost - this.initialFee
    this.totalMounthlyPayment = Math.floor((this.totalAmountOfCredit + (((this.totalAmountOfCredit / 100) * this.percent) / 12) * this.creditTerm) / this.creditTerm)
    this.totalRecommendedIncome = this.totalMounthlyPayment + ((this.totalMounthlyPayment / 100) * 35)

    if (this.totalCost < this.initialFee) {
      this.initialFee = this.totalCost
    }
  },

  name: 'App',
  components: {
    ElSlider,
    AppSelectBankItem
  }
}
</script>

<style lang="scss">
@import '@/assets/styles/main.scss';
// @import './assets/styles/element-ui.css';

.content {
  padding-top: 80px;
  display: flex;

  .content__column {
    padding: 0 20px;

    &:first-child {
      flex: 1 1 100%;
      border-right: 2px solid #e5e5e5;
    }

    &:last-child {
      flex: 1 0 250px;
    }
  }

  &-inputs {
    margin: 0px 0px 90px 0px;
  }
}

.show-items {
  margin: 80px 0px 0px 0px;
  .show-item {
    margin: 0px 0px 25px 0px;
    &__label {
      font-size: 18px;
      margin: 0px 0px 15px 0px;
    }
    &__result {
      font-size: 24px;
    }
  }
}

.content__bank-selection {
  border-top: 2px solid #e5e5e5;
  padding: 30px 0px 0px 0px;
  .select-bank__title {
    font-size: 30px;
    margin: 0px 0px 25px 0px;
  }
}

.el-slider {
  padding: 0 10px;

  .el-slider__runway {
    height: 20px; // height for main line
    margin: 23px 0 16px 0;
    background-color: #e4e7ed;
    border-radius: 15px;
    position: relative;
    cursor: pointer;
    vertical-align: middle;
    box-shadow: 3px 3px 4px rgba(0, 0, 0, 0.25);

    .el-slider__bar {
      height: 20px; // height for active line
      background-color: #1adf2e;
      border-radius: 15px;
      position: absolute;
    }

    .el-slider__button-wrapper {
      height: 36px;
      width: 36px;
      position: absolute;
      z-index: 1;
      top: -8px;
      transform: translateX(-50%);
      background-color: #fff;
      text-align: center;
      border-radius: 50%;
      box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.4);

      .el-slider__button {
        display: inline-block;
        width: 16px;
        height: 16px;
        background-color: #1adf2e;
        vertical-align: middle;
        border-radius: 50%;
        box-sizing: border-box;
        transition: 0.2s;
      }

      &:after {
        display: inline-block;
        content: '';
        height: 100%;
        vertical-align: middle;
      }
    }

    .el-slider__stop.el-slider__marks-stop {
      position: absolute;
      height: 26px;
      width: 26px;
      border-radius: 50%;
      background-color: #e5e5e5;
      transform: translate(-50%, -11%);

      &.done {
        background: #1adf2e;
      }

      &:after {
        content: '';
        width: 12px;
        height: 12px;
        background: #fff;
        display: inline-block;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        border-radius: 50%;
      }
    }

    .el-slider__stop {
      background-color: #000;
    }

    .el-slider__marks {
      .el-slider__marks-text {
        position: absolute;
        transform: translateX(-50%);
        font-size: 14px;
        color: #000;
        margin-top: 40px;
      }
    }
  }
}
</style>
