<template>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.6.0/css/all.min.css"/>
  <link rel="icon" href="/public/favicon.ico" type="image/icon type"/>
  <div id="app">
    <router-view/>
    <div class="full-body">
      <div class="header">
        <ul>
          <li>
            <a href="/"> <i class="fa-solid fa-house"></i> Anasayfa </a>
          </li>
          <li>
            <a v-if="isLogged == 1" href="/history"> <i class="fa-solid fa-ghost"></i> Geçmiş </a>
          </li>
          <li>
            <a v-if="isLogged == 1" href="/settings"> <i class="fa-solid fa-user-gear"></i> Ayarlar </a>
          </li>
          <li>
            <a v-if="isLogged == 1" @click="logout" href="/">
              <i class="fa-solid fa-door-open"></i> Çıkış
            </a>
          </li>
          <abbr title="Giriş Yap">
            <button v-if="isLogged == 0" @click="navigateToLogin"> Giriş Yap </button>
          </abbr>
          <abbr title="Kaydol">
            <button v-if="isLogged == 0" @click="navigateToRegister"> Kaydol </button>
          </abbr>
          <abbr title="Admin Girişi">
            <button v-if="isAdmin == 1" @click="navigateToAdmin">Admin Girişi</button>
          </abbr>
        </ul>
      </div>
      <div class="islemler">
        <div id="randomsayi1">
          <label for="first-number">İlk Sayı</label>
          <input type="number" id="first-number" :value="firstNumber" readonly />
        </div>
        <div class="zorluk">
          <template v-for="(option, index) in zorlukOptions" :key="index">
            <div class="checkbox-container">
              <input :id="'derece' + (index + 1)" type="radio" name="zorluk" :value="option.value" v-model="selectedDifficulty" @change="updateNumbers"/>
              <label :for="'derece' + (index + 1)">
                {{ option.label }}
              </label>
            </div>
          </template>
        </div>
        <div id="randomsayi2">
          <label for="second-number">İkinci Sayı</label>
          <input type="number" id="second-number" :value="secondNumber" readonly />
        </div>
        <div class="islem">
          <label for="operation-select">Bir işlem seçin</label>
          <select id="operation-select" v-model="selectedOperation" :disabled="operationLocked">
            <option value="add">Toplama</option>
            <option value="subtract">Çıkarma</option>
            <option value="multiply">Çarpma</option>
            <option value="divide">Bölme</option>
          </select>
        </div>
        <div id="sonuc">
          <label for="result">Sonuç</label>
          <input type="number" id="result" v-model="result" @keyup.enter="checkResult"/>
          <div id="kontrol">
            <abbr title="Sonucu Kontrol Et">
              <button @click="checkResult">Sonucu Kontrol Et</button>
            </abbr>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "Home",
  computed: {
    isLogged() {
      return this.$store.getters.isLogged;
    },
    isAdmin() {
      return this.$store.getters.isAdmin;
    }
  },
  data() {
    return {
      firstNumber: 0,
      secondNumber: 0,
      selectedOperation: "add",
      operationLocked: false,
      result: "",
      nickname: "YigithanFirat",
      sorusayisi: 10,
      selectedDifficulty: "easy",
      quizCompleted: false,
      questionCount: 0,
      totalQuestions: 10,
      zorlukOptions: [
        { value: "easy", label: "Kolay" },
        { value: "medium", label: "Orta" },
        { value: "hard", label: "Zor" },
      ],
    };
  },

  methods: {
    async logout() {
      try {
        const response = await axios.post("http://localhost:3000/logout", {
          userId: 1,
        });
        if (response.data && response.data.message === "Çıkış işlemi başarılı.") {
          alert("Başarıyla çıkış yaptınız.");
          this.$store.dispatch("logout");
        } else {
          alert(response.data.error || "Çıkış işlemi başarısız. Tekrar deneyin.");
        }
      } catch (error) {
        console.error("Hata Detayı:", error.response?.data || error.message || error);
        alert("Sunucuya bağlanırken bir hata oluştu. Lütfen tekrar deneyin.");
      }
    },

    generateRandomNumbers() {
      switch (this.selectedDifficulty) {
        case "easy":
          this.firstNumber = Math.floor(Math.random() * 10) + 1;
          this.secondNumber = Math.floor(Math.random() * 10) + 1;
          break;
        case "medium":
          this.firstNumber = Math.floor(Math.random() * 90) + 10;
          this.secondNumber = Math.floor(Math.random() * 90) + 10;
          break;
        case "hard":
          this.firstNumber = Math.floor(Math.random() * 900) + 100;
          this.secondNumber = Math.floor(Math.random() * 900) + 100;
          break;
      }
      this.result = "";
    },

    async checkResult() {
      let expectedResult;
      switch (this.selectedOperation) {
        case "add":
          expectedResult = this.firstNumber + this.secondNumber;
          break;
        case "subtract":
          expectedResult = this.firstNumber - this.secondNumber;
          break;
        case "multiply":
          expectedResult = this.firstNumber * this.secondNumber;
          break;
        case "divide":
          if (this.secondNumber === 0) {
            alert("Bir sayıyı sıfıra bölemezsiniz!");
            return;
          }
          if (this.firstNumber === 0 && this.secondNumber === 0) {
            alert("0/0 belirsizdir!");
            return;
          }
          expectedResult = this.firstNumber / this.secondNumber;
          break;
      }

      if (Number(this.result) === expectedResult) {
        this.questionCount++;
        if (this.questionCount === 1) {
          this.startTime = Date.now();
        }
        if (this.questionCount === this.sorusayisi) {
          this.endTime = Date.now();
          const totalTime = Math.floor((this.endTime - this.startTime) / 1000);
          await this.saveResults(totalTime);
          alert(`Tebrikler! Toplam süre: ${totalTime} saniye.`);
          this.$router.push("/");
        } else {
          this.generateRandomNumbers();
        }
      } else {
        alert("Yanlış cevap!");
      }
    },

    async saveResults(totalTime) {
      if (!this.isLogged) {
        alert("Sonuçları kaydetmek için giriş yapmalısınız.");
        return;
      }

      const puan = this.questionCount;
      const payload = {
        zorluk: this.selectedDifficulty,
        sorusayisi: this.sorusayisi,
        nickname: this.nickname,
        puan: puan,
        toplamSure: totalTime,
      };

      console.log("Gönderilen Payload:", payload);
      try {
        const response = await axios.post("http://localhost:3000/saveResults", payload);
        if (response.status === 200) {
          console.log("Sonuç başarıyla kaydedildi:", response.data);
          alert("Sonuç başarıyla kaydedildi.");
        } else {
          console.error("Hata:", response.data.message);
          alert("Sonuç kaydedilemedi: " + response.data.message);
        }
      } catch (error) {
        console.error("Axios hatası:", error);
        alert("Sonuç kaydedilirken bir hata oluştu.");
      }
    },

    updateNumbers() {
      this.generateRandomNumbers();
    },

    navigateToLogin() {
      this.$router.push("/login");
    },

    navigateToAdmin() {
      this.$router.push("/adminlogin");
    },

    navigateToRegister() {
      this.$router.push("/register");
    },

    resetQuiz() {
      this.quizCompleted = false;
      this.questionCount = 0;
      this.generateRandomNumbers();
    },
  },

  mounted() {
    const operation = this.$route.query.operation;
    if (operation) {
      this.selectedOperation = operation;
      this.operationLocked = true;
    }
    this.generateRandomNumbers();
  },
};
</script>

<style>

@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap');

*
{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Roboto', sans-serif;
}

body, html 
{
  width: 100%;
  height: 100%;
}

#app 
{
  width: 100%;
  height: 100%;
}

.full-body 
{
  position: relative;
  width: 100%;
  height: 100vh;
  background-image: url("/src/assets/logo.png");
  background-position: center;
  background-size: cover;
  background-attachment: fixed;
  background-repeat: no-repeat;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
}

.header 
{
  background-color: #112479;
  width: 100%;
  padding: 10px 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.header ul 
{
  list-style-type: none;
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin: 0;
  padding: 0;
}

.header ul li a 
{
  display: inline-block;
  padding: 10px 20px;
  text-decoration: none;
  font-size: 18px;
  color: #fafafa;
  font-weight: 500;
  letter-spacing: 1px;
  transition: color 0.3s, background-color 0.3s;
  border-radius: 4px;
}

.header ul li a:hover 
{
  background-color: black;
}

.header abbr 
{
  text-decoration: none;
  cursor: pointer;
}

.header abbr button 
{
  border: none;
  padding: 8px 16px;
  font-size: 14px;
  font-weight: 600;
  color: #fafafa;
  background-color: #1a73e8;
  border-radius: 30px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: background-color 0.3s, transform 0.3s;
}

.header abbr button:hover 
{
  background-color: #0c5bd5;
  transform: scale(1.05);
}

.zorluk-container 
{
  margin-top: 50px;
  padding: 20px;
  background-color: rgba(17, 36, 121, 0.8);
  border-radius: 12px;
  width: 90%;
  max-width: 400px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.zorluk-container h3 
{
  color: #fafafa;
  font-size: 20px;
  margin-bottom: 20px;
  font-weight: 600;
}

.zorluk 
{
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
}

.zorluk .checkbox-container 
{
  display: flex;
  align-items: center;
  gap: 8px;
}

.zorluk label 
{
  font-size: 16px;
  color: #fafafa;
  font-weight: 600;
}

.islemler 
{
  margin-top: 30px;
  text-align: center;
  padding: 20px;
  background-color: rgba(17, 36, 121, 0.8);
  border-radius: 12px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.islemler label 
{
  color: #fafafa;
  font-weight: bold;
  display: block;
  margin-bottom: 10px;
  letter-spacing: 0.5px;
}

.islemler input, .islemler select 
{
  width: 100%;
  padding: 10px;
  font-size: 14px;
  margin-bottom: 15px;
  border: 2px solid #2641FF;
  border-radius: 4px;
  background-color: #f9f9f9;
  outline: none;
  transition: border-color 0.3s;
}

.islemler input:focus, .islemler select:focus 
{
  border-color: #0c5bd5;
}

.islemler button 
{
  border: none;
  padding: 10px 20px;
  font-size: 14px;
  font-weight: 600;
  color: #fafafa;
  background-color: #1a73e8;
  border-radius: 30px;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.3s;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.islemler button:hover 
{
  background-color: #0c5bd5;
  transform: scale(1.05);
}

.islemler #sonuc abbr button 
{
  width: auto;
  padding: 10px 20px;
}

.islemler input[readonly] 
{
  background-color: #e9ecef;
  color: #495057;
  cursor: not-allowed;
}

</style>