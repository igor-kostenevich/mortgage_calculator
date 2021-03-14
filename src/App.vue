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
              :max="228"
              :min="12"
              :step="12"
              :show-tooltip="false"
              label="Срок кредита"
              :marks="dataMarks.marks"
              :class="[{ 'el-slider-marks-mobile': handlerResize }]"
              id="el-slider-marks"
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
            >{{ bank.name }}</app-select-bank-item
          >
        </div>
      </div>
      <div class="content__column">
        <div class="show-items">
          <div class="show-item">
            <div class="show-item__label">Сумма кредита</div>
            <div class="show-item__result">
              {{ currency(totalAmountOfCredit) }}
            </div>
          </div>
          <div class="show-item">
            <div class="show-item__label">Ежемесячный платеж</div>
            <div class="show-item__result">
              {{ currency(totalMounthlyPayment) }}
            </div>
          </div>
          <div class="show-item">
            <div class="show-item__label">Рекомендуемый доход</div>
            <div class="show-item__result">
              {{ currency(totalRecommendedIncome) }}
            </div>
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
          216: '216',
          228: '228'
        },
        arrMarksItems: [],
        dataMarksInfo: []
      },
      selectBanks: [
        { name: 'Приватбанк', rate: 7, isActive: true },
        { name: 'Альфа-банк', rate: 7.5, isActive: false },
        { name: 'Мегабанк', rate: 6.9, isActive: false },
        { name: 'ОТП банк', rate: 7.1, isActive: false },
        { name: 'Ощадбанк', rate: 7.3, isActive: false }
      ],

      currency,
      windowWidth: window.innerWidth,
    }
  },

  methods: {
    isActiveBank (bank) {
      if (!bank.isActive) {
        const oldActiveBank = this.selectBanks.find(item => item.isActive)
        oldActiveBank.isActive = false
        bank.isActive = true
        this.percent = bank.rate
      }
    }
  },

  computed: {
    handlerResize () {
      return this.windowWidth < 550
    }
  },

  mounted () {
    this.dataMarks.arrMarksItems.push(...document.querySelectorAll('.el-slider__marks-stop'))

    this.dataMarks.dataMarksInfo = this.dataMarks.arrMarksItems.map(item => ({ item }))
    const arrMarksValues = Object.values(this.dataMarks.marks).map(item => +item)
    this.dataMarks.dataMarksInfo.forEach((item, idx) => item.marksValue = arrMarksValues[idx])

    window.onresize = e => this.windowWidth = window.innerWidth
  },

  watch: {
    creditTerm (elem) {
      this.dataMarks.dataMarksInfo.forEach(item => elem >= item.marksValue ? item.item.classList.add('done') : item.item.classList.remove('done'))
    }
  },

  updated () {
    // calculator logic
    this.totalAmountOfCredit = this.totalCost - this.initialFee
    this.totalMounthlyPayment = Math.round(
      (this.totalAmountOfCredit + (((this.totalAmountOfCredit / 100) * this.percent) / 12) * this.creditTerm) / this.creditTerm
    )
    this.totalRecommendedIncome = this.totalMounthlyPayment + (this.totalMounthlyPayment / 100) * 35

    if (this.totalCost < this.initialFee) {
      this.initialFee = this.totalCost
    }



    // old breakpoint logic for creditTerm input, it's not used
    
    // if (document.querySelector('#el-slider-marks')) {
      // const ss = Object.values(this.dataMarks.marks).map(item => console.log(+item))
      // const btnSlider = document.querySelector('.el-slider-marks .el-slider__button-wrapper')
      // // const getStyleButtonDistance = parseFloat(getComputedStyle(btnSlider).left)
      // this.abc = parseFloat(getComputedStyle(btnSlider).left)
      // const arrMarksDone = this.dataMarks.arrMarksItems.filter((item, idx) => {
      //   if (idx > this.dataMarks.dataMarksInfo.indexOf(this.abc)) {
      //     item.classList.remove('done')
      //   }
      //   return idx <= this.dataMarks.dataMarksInfo.indexOf(this.abc)
      // })
      // arrMarksDone.forEach(item => item.classList.add('done'))
    // }
  },

  name: 'App',
  components: {
    ElSlider,
    AppSelectBankItem
  }
}
</script>

<style lang="scss">
@import '@/assets/styles/scss/main.scss';

.content {
  padding-top: 80px;
  display: flex;

  @media screen and (max-width: 550px) {
    padding-top: 50px;
  }

  &__column {
    padding: 0 20px;

    @media screen and (max-width: 550px) {
      padding: 0 10px;
    }

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

  @media (max-width: 550px) {
    margin: 30px 0px 0px 0px;
    border-top: 2px solid #e5e5e5;
    padding-top: 30px;
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

@media (max-width: 860px) {
  .content {
    flex-direction: column;

    &__column:first-child {
      border: none;
    }
  }
}
</style>
