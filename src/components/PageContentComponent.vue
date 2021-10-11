<template>
  <div>
    <div v-if="showLoader" id="loader">
      <div class="lds-ring">
        <div></div>
        <div></div>
        <div></div>
        <div></div>
      </div>
    </div>

    <div class="navbar">
      <div class="navbar_left_mask"></div>
      <h1>Editor panel</h1>
      <hr class="top_header_separator" />
    </div>

    <div class="layout">
      <div class="left_section">
        <div class="triangle"></div>
        <div class="left_section_header_container">
          <div class="left_section_header">
            Insert the text to be transformed below
          </div>
          <button class="load_default_text_btn" @click="LoadDefaultText">
            Default text
          </button>
          <hr class="left_section_header_separator" />
        </div>
        <form action="#">
          <div class="row">
            <textarea
              v-model="textArea"
              ref="TextArea"
              placeholder="Insert text here or paste default clicking the button"
              class="text_area_enter"
            ></textarea>
            <div class="left_section_button">
              <button type="button" @click="getText">Action</button>
            </div>
          </div>
        </form>
        <div id="error_message">{{ this.error }}</div>
      </div>
      <div class="right_section">
        <div class="reight_section_header">Transformation result</div>
        <div id="show_results">{{ this.showText }}</div>
        <div class="right_section_button">
          <button @click="clearAll">Clear</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "PageContentComponent",
  data() {
    return {
      tagsArray: [], // Array of tags found in the original text
      ApiNameArray: [], // Array of all currency names from API
      error: "",
      textArea: "",
      showLoader: false,
      showText: "",
      textBeforeShow: "", // Temporary text variable - replaces tags with names here
      defaultText: 'In 1998, Wei Dai published a description of "b-money", characterized as ananonymous, distributed electronic cash system.[Shortly thereafter, Nick Szabodescribed bit gold. Like {{ Name/BTC }} and other cryptocurrencies self wouldfollow it, bit gold (not to be confused with the later gold-based exchange, {{ Name/BITGOLD }}) was described as an electronic currency system which requiredusers to complete a proof of work function with solutions being cryptographicallyput together and published. A currency system based on a reusable proof of workwas later created by Hal Finney who followed the work of Dai and Szabo.The first decentralized cryptocurrency, {{ Name/BTC }}, was created in 2009 bypseudonymous developer Satoshi Nakamoto. It used SHA-256, a cryptographichash function, as its proof-of-work scheme. In April 2011, {{ Name/NMC }} wascreated as an attempt at forming a decentralized DNS, which would make internetcensorship very difficult. Soon after, in October 2011, {{ Name/LTC }} was released. Itwas the first successful cryptocurrency to use scrypt as its hash function instead ofSHA-256. Another notable cryptocurrency, {{ Name/PPC }} was the first to use aproof-of-work/proof-of-stake hybrid.',
    };
  },
  methods: {
    getText() {
      // Reset all variables
      this.tagsArray = [];
      this.ApiNameArray = [];
      this.error = "";

      if (this.textArea === "") {
        return (this.error = "Text not detected.");
      } else {
        this.error = "";
      }

      // Get tags from the text
      let extractTags = this.textArea.match(/{{.*?}}/g);
      if (!extractTags) {
        return (this.error = "No tags found.");
      }

      this.showLoader = true;

      for (let val of extractTags) {

        // Get symbol from tag
        val = val.replace(/({{ Name\/| }})/g, "");

        // Letter validation - only uppercase letters allowed
        if (val == val.toLowerCase()) {
          this.showLoader = false;
          return (this.error = "Some currency symbols are lowercase.");
        }

        this.tagsArray.push({ SymbolName: val, OriginalText: val });
      }

      // Call Name() function
      this.tagsArray.forEach((element) => {
        this.Name(element.SymbolName);
      });
    },
    Name(param) {
      let params = {
        q: param,
        c: "currencies",
        modifier: "symbol_search",
        limit: 1,
      };

      let query = Object.keys(params)
        .map((k) => encodeURIComponent(k) + "=" + encodeURIComponent(params[k]))
        .join("&");

      let url = "https://api.coinpaprika.com/v1/search?" + query;

      let self = this;

      fetch(url)
        .then((data) => data.json())
        .then((response) => {

          if (response.error) {
            return (self.error = response.error);
          }

          // Get currency names from API
          self.ApiNameArray.push({
            ID: response.currencies[0].id,
            Name: response.currencies[0].name,
            SymbolToRemove: "{{ Name/" + response.currencies[0].symbol + " }}",
          });
         
        })
        .then(() => {
          let transformedText = "";

          // Replace the original text with the names
          self.textBeforeShow = self.textArea;

          self.ApiNameArray.forEach(function (value) {
            transformedText = self.textBeforeShow.replace(
              value.SymbolToRemove,
              value.Name
            );
            self.textBeforeShow = transformedText;
          });

          setTimeout(() => {
            if (self.error === "") {
              self.showText = transformedText;
            } else {
              self.showText = "";
            }
            self.showLoader = false;
          }, 1000);

        })
        .catch(() => {
          if (self.error !== "") {
            self.error = self.error;
          } else {
            self.error = "The inserted text contains errors.";
            self.showText = "";
          }

          setTimeout(() => {
            self.showLoader = false;
          }, 1000);
        });
    },
    LoadDefaultText() {
      // Get the default task text into the textarea
      this.textArea = this.defaultText;
    },
    clearAll() {
      // Reset all properties - called with the "Clear" button
      this.textArea = "";
      this.showText = "";
      this.error = "";
      this.tagsArray = [];
      this.ApiNameArray = [];
    },
  },
};
</script>

<style scoped>
.left_section {
  background-color: #395477;
  width: 50%;
  min-height: calc(100vh - 70px);
  border-top-right-radius: 30px;
  border-bottom-right-radius: 30px;
  box-shadow: 3px 6px 6px 2px rgb(150, 150, 150);
}
.row {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
}
.text_area_enter {
  width: calc(80% - 60px);
  max-width: 1000px;
  margin: auto;
  min-height: 50vh;
  border-radius: 20px;
  padding: 15px 30px;
  border: none;
  background-color: #4b6383;
  color: #f1f7ff;
  font-size: 14px;
  font-family: "Segoe UI", sans-serif;
  transition: 0.3s;
}
.text_area_enter:focus {
  outline: none;
  transform: scale(1.02);
}
.left_section_header_container {
  width: 80%;
  max-width: 1000px;
  height: 55px;
  margin: 20px auto;
  display: flex;
  justify-content: space-between;
  position: relative;
}
.left_section_header {
  font-size: 21px;
  line-height: 60px;
  color: #f4f4f4;
  font-weight: 600;
  padding-right: 10px;
  transition: 0.3s;
  text-align: left;
}
@media (max-width: 960px) {
  .left_section_header {
    font-size: 18px;
  }
}
.load_default_text_btn {
  border-radius: 50%;
  height: 55px;
  width: 55px;
  border: none;
  font-weight: 600;
  font-size: 11px;
  color: #4c4c4c;
  cursor: pointer;
  background-color: #ffffff;
  transition: 0.3s;
}
.load_default_text_btn:hover {
  transform: scale(1.03);
}
.left_section_header_separator {
  width: 70%;
  position: absolute;
  bottom: -10px;
  left: 7%;
  color: #ffffff;
}
.left_section_button,
.right_section_button {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-top: 30px;
}
.left_section_button > button,
.right_section_button > button {
  width: 140px;
  padding: 10px;
  border-radius: 10px;
  border: none;
  font-family: "Segoe UI", sans-serif;
  font-weight: 600;
  cursor: pointer;
}
.right_section {
  width: calc(45% - 40px);
  padding: 15px 20px;
  margin: 0 auto;
}
.reight_section_header,
#show_results {
  background-color: teal;
  width: calc(100% - 40px);
  height: 25px;
  border-radius: 7px;
  padding: 10px 20px;
  color: #ffffff;
  font-weight: 600;
  line-height: 25px;
  text-align: left;
}
#show_results {
  height: 60vh;
  overflow-y: auto;
  margin-top: 20px;
  font-weight: 500;
  background-color: #ffffff;
  color: rgb(73, 73, 73);
  text-align: left;
  box-shadow: 0px 1px 6px 0px rgb(173, 173, 173);
}
.right_section_button > button {
  background-color: #a83838;
  color: #ffffff;
}
@media (max-width: 882px) {
  .left_section {
    width: 95%;
    margin-bottom: 30px;
    min-height: unset;
    padding-bottom: 25px;
  }
  .right_section {
    width: 90%;
    padding-bottom: 30px;
  }
  #show_results {
    min-height: 60vh;
    height: auto;
    overflow: auto;
  }
}
@media (max-width: 480px) {
  .left_section_header {
    line-height: 25px;
  }
  .text_area_enter {
    padding: 10px;
    width: calc(80% - 20px);
  }
  .top_header_separator {
    width: 65%;
  }
}
#loader {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 7;
  background-color: rgba(187, 187, 187, 0.7);
  display: flex;
}
.lds-ring {
  display: inline-block;
  position: relative;
  width: 80px;
  height: 80px;
  margin: auto;
}
.lds-ring div {
  box-sizing: border-box;
  display: block;
  position: absolute;
  width: 64px;
  height: 64px;
  margin: 8px;
  border: 8px solid #fff;
  border-radius: 50%;
  animation: lds-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
  border-color: #fff transparent transparent transparent;
}
.lds-ring div:nth-child(1) {
  animation-delay: -0.45s;
}
.lds-ring div:nth-child(2) {
  animation-delay: -0.3s;
}
.lds-ring div:nth-child(3) {
  animation-delay: -0.15s;
}
@keyframes lds-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
#error_message {
  font-weight: 600;
  width: 80%;
  margin: 20px auto;
  text-align: center;
  color: #5c0909;
  text-decoration: underline;
}
</style>
