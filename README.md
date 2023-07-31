//отпрвка в строке URL на https://allrightcasino.nascms.co/api/bonus/info/uuids={{Campagin_true}}

// Функция для отправки асинхронных запросов с промисами
function sendRequestAsync(url) {
  return new Promise((resolve, reject) => {
    pm.sendRequest(url, (err, response) => {
      if (err) {
        reject(err);
      } else {
        resolve(response);
      }
    });
  });
}

// Отправляем первый запрос к Google Sheets API
var Cell = pm.environment.get("Cell");
var Letter1 = pm.environment.get("Letter1");
var Letter2 = pm.environment.get("Letter2");
sendRequestAsync(`https://sheets.googleapis.com/v4/spreadsheets/1-THQr1Y0Rkm_JnUurjmjrvntCCg_rHdY9YkSSSja8xo/values:batchGet?ranges=AllRight!${Letter1}${Cell}:${Letter2}${Cell}&key=AIzaSyCNMwgRQjntdquqKkb52r_2jcgpMQb0LXw`)
  .then((response) => {
    const jsonData = response.json();
    if (jsonData && jsonData.valueRanges && jsonData.valueRanges.length > 0 && jsonData.valueRanges[0].values && jsonData.valueRanges[0].values.length > 0) {
      const values = jsonData.valueRanges[0].values[0];
      // Сохраняем значения в переменные окружения
      pm.environment.set("GameName_true", values[0]);
      pm.environment.set("RestrictedCountries_true", values[1]);
      pm.environment.set("emptyValue", values[2]);
      pm.environment.set("countries2", values[3]);
      pm.environment.set("countries3", values[4]);
      pm.environment.set("provider/BO_true", values[5]);
      pm.environment.set("additionalTags_true", values[6]);
      pm.environment.set("Wager_true", values[7]);
      pm.environment.set("AmountFreeSpin_true", values[8]);
      pm.environment.set("Bonus_Valid_time_true", values[9]);
      pm.environment.set("freeSpinPrice/BO_true", values[10]);
      pm.environment.set("Campagin_true", values[values.length - 1]);

      // Возвращаем значения для использования в следующем обещании
      return values[values.length - 1];
    } else {
      throw new Error("Ошибка: Некорректные данные или отсутствие значений в ответе.");
    }
  })
  .then((campaign) => {
    // Отправляем второй запрос с использованием полученного значения "campaign"
    return sendRequestAsync(`https://allrightcasino.nascms.co/api/bonus/info/${campaign}/CAMPAIGN`);
  })
  .then((response) => {
    var responseBody = response.json();
    var range = responseBody.fulfillments[0].range;
    var maxBet = responseBody.templates[0].maxBet;
    var maxBetSport = responseBody.templates[0].maxBetSport;
    var maxGrantAmount = responseBody.templates[0].maxGrantAmount;
    var name = responseBody.name;
    var state = responseBody.state;
    var period = responseBody.additionalInfo.resetSetting && responseBody.additionalInfo.resetSetting.period;
    var action = responseBody.additionalInfo.resetSetting && responseBody.additionalInfo.resetSetting.action;
    var targetType = responseBody.additionalInfo.targetType;
    var optInPeriod = responseBody.additionalInfo.optInTimeout.optInPeriod;
    var optInPeriodTimeUnit = responseBody.additionalInfo.optInTimeout.optInPeriodTimeUnit;
    var campaignPeriod = responseBody.additionalInfo.campaignPeriod;
    var fulfillmentPeriodInfo = responseBody.additionalInfo.fulfillmentPeriodInfo;
    var provider = responseBody.additionalInfo.templates[0].provider;
    var game = responseBody.additionalInfo.templates[0].game;
    var freeSpinsAmount = responseBody.additionalInfo.templates[0].freeSpinsAmount;
    var freeSpinPrice = responseBody.additionalInfo.templates[0].freeSpinPrice;
    var percentage = responseBody.additionalInfo.templates[0].wageringRequirement[0].requirement.percentage;

    // Сохранение значений в переменную окружения без форматирования
    pm.environment.set("percentage/BO", JSON.stringify(percentage));
    pm.environment.set("deposit/BO", JSON.stringify(range));
    pm.environment.set("maxBet/BO", JSON.stringify(maxBet));
    pm.environment.set("maxBetSport/BO", JSON.stringify(maxBetSport));
    pm.environment.set("maxGrantAmount/BO", JSON.stringify(maxGrantAmount));
    pm.environment.set("name/BO", JSON.stringify(name));
    pm.environment.set("state/BO", JSON.stringify(state));
    pm.environment.set("period/BO", JSON.stringify(period));
    pm.environment.set("action/BO", JSON.stringify(action));
    pm.environment.set("targetType/BO", JSON.stringify(targetType));
    pm.environment.set("optInPeriod/BO", JSON.stringify(optInPeriod));
    pm.environment.set("optInPeriodTimeUnit/BO", JSON.stringify(optInPeriodTimeUnit));
    pm.environment.set("campaignPeriod/BO", JSON.stringify(campaignPeriod));
    pm.environment.set("fulfillmentPeriodInfo/BO", JSON.stringify(fulfillmentPeriodInfo));
    pm.environment.set("provider/BO", JSON.stringify(provider));
    pm.environment.set("game/BO", JSON.stringify(game));
    pm.environment.set("freeSpinsAmount/BO", JSON.stringify(freeSpinsAmount));
    pm.environment.set("freeSpinPrice/BO", JSON.stringify(freeSpinPrice));
  })
  .catch((error) => {
    console.log("Ошибка:", error);
  });

var MyData = JSON.parse(responseBody);

postman.setEnvironmentVariable("allowedCountries", MyData[0].allowedCountries);
postman.setEnvironmentVariable("restrictedCountries", MyData[0].restrictedCountries);
postman.setEnvironmentVariable("additionalTags", MyData[0].additionalTags);
postman.setEnvironmentVariable("sourceTags", MyData[0].sourceTags);

var allowed = pm.environment.get("allowedCountries").split(",");
var restrict = pm.environment.get("restrictedCountries").split(",");
var trueCountries = allowed.concat(restrict);

// Преобразуем массив trueCountries в строку с запятыми
var trueCountriesString = trueCountries.join(",");

// Удаляем последнюю запятую из строки
trueCountriesString = trueCountriesString.replace(/,$/, "");

postman.setEnvironmentVariable("Countries_true", trueCountriesString);

const languages = ["pt", "tr", "ru", "no", "fi", "de", "fr", "pl", "en", "es", "sv", "ja"];

const translations = MyData[0].translations.title;
languages.forEach((lang) => {
  const variableName = `translations/title/${lang}`;
  postman.setEnvironmentVariable(variableName, translations[lang]);
});

const translations1 = MyData[0].translations.description;
languages.forEach((lang) => {
  const variableName = `description/translations/${lang}`;
  postman.setEnvironmentVariable(variableName, translations1[lang]);
});

const translationswihoutdata = MyData[0].translations.description;
languages.forEach((lang) => {
  const variableName = `description/translationsWithoutData/${lang}`;
  postman.setEnvironmentVariable(variableName, translationswihoutdata[lang].replace(/{[^}]+}/g, ""));
});

for (var i = 0; i < languages.length; i++) {
  var language = languages[i];
  var descriptionKey = "description/" + language + "_full_String";
  var descriptionValue = MyData[0].translations.description[language].replace(/\s/g, '');
  postman.setEnvironmentVariable(descriptionKey, descriptionValue);
}

var tests = [
  "DepositLVL1_true_RegularBonuses",
  "DepositLVL2_true_RegularBonuses",
  "DepositLVL3_true_RegularBonuses",
  "DepositLVL1_true_RegularBonuses(CZE,ESP.etc)",
  "DepositLVL2_true_RegularBonuses(CZE,ESP.etc)",
  "DepositLVL3_true_RegularBonuses(CZE,ESP.etc)",
  "DepositLVL1_true_RegularBonuses(POL)",
  "DepositLVL2_true_RegularBonuses(POL)",
  "DepositLVL3_true_RegularBonuses(POL)"
];
var skippedTests = 0;

tests.forEach(function (test) {
  languages.forEach(function (language) {
    try {
      var dep_true = pm.environment.get(`description/${language}_full_String`);
      if (pm.expect(dep_true).to.include(pm.environment.get(test))) {
        pm.test(`${test} for ${language} language is correct`, function () {
          pm.expect(dep_true).to.include(pm.environment.get(test));
        });
      }
    } catch (e) {
      skippedTests++;
      pm.test.skip(`${test} for ${language} language is not correct`, function () {
        pm.expect(dep_true).to.include(pm.environment.get(test));
      });
    }
  });
});

if (skippedTests === tests.length * languages.length) {
  pm.test("Deposit from CMS are not correct", function () {
    pm.expect.fail("Deposit from CMS are not correct");
  });
}

var dep_true1 = pm.environment.get("deposit/BO");
var tests1 = [
  "DepositLVL1_true_RegularBonuses/BO",
  "DepositLVL2_true_RegularBonuses/BO",
  "DepositLVL3_true_RegularBonuses/BO",
  "DepositLVL1_true_RegularBonuses(CZE,ESP.etc)/BO",
  "DepositLVL2_true_RegularBonuses(CZE,ESP.etc)/BO",
  "DepositLVL3_true_RegularBonuses(CZE,ESP.etc)/BO",
  "DepositLVL1_true_RegularBonuses(POL)/BO",
  "DepositLVL2_true_RegularBonuses(POL)/BO",
  "DepositLVL3_true_RegularBonuses(POL)/BO"
];
var skippedTests1 = 0;

tests1.forEach(function(test1) {
  try {
    if (pm.expect(dep_true1).to.include(pm.environment.get(test1))) {
      pm.test(`${test1} are correct`, function() {
        pm.expect(dep_true1).to.include(pm.environment.get(test1));
      });
    }
  } catch (e) {
    skippedTests1++;
    pm.test.skip(`${test1} are not correct`, function() {
      pm.expect(dep_true1).to.include(pm.environment.get(test1));
    });
  }
});

if (skippedTests1 === tests1.length) {
  pm.test("Deposit from BO or CMS are not correct", function() {
    pm.expect.fail("Deposit from BO or CMS are not correct");
  });
}

let originalString = pm.environment.get("GameName_true");
let regex = /\d+/;
let matches = originalString.match(regex);
if (matches && matches.length > 0) {
  let extractedNumber = matches[0];
  pm.environment.set("extractedNumberVariable", extractedNumber);
}

var Countries = pm.environment.get("Countries_true").replace(/\s/g, "").split(",");
var CountriesTrue = pm.environment.get("RestrictedCountries_true").replace(/\s/g, "").split(",");
var setCountries = new Set(Countries);
var setCountriesTrue = new Set(CountriesTrue);
pm.test("Restricted and Allowed Countries are correct", function () {
    pm.expect(Countries).to.eql(CountriesTrue);
});

pm.test("AdditionalTags are correct", function () {
pm.expect(pm.environment.get("additionalTags")).to.eql((pm.environment.get("additionalTags_true")))});

pm.test("SourseTag are correct", function () {
pm.expect(pm.environment.get("sourceTags")).to.eql((pm.environment.get("sourceTags_true")))});

var freespinsKey = [["rodadas grátis", "jogadas gratuitas", "giros", "giros grátis", "giros grátis"], ["freespini", "adet"], "фриспинов", "free spins", "free spins", "Freispiele", "tours gratuits", ["darmowych spinów", "free spinów", "darmowe spiny"],  "free spins", ["tiradas gratis", "giros gratis"], ["gratissnurr", "gratisspinn"], ["フリースピン", "で", "フリースピン","フリースピン"]];

var AmountFsTrue = parseInt(pm.environment.get("AmountFreeSpin_true"));
languages.forEach((language, index) => {
  var FsPlusNameTrue = `amount/fs/${language}`;
  var FsPlusName = `${AmountFsTrue} ${Array.isArray(freespinsKey[index]) ? freespinsKey[index][0] : freespinsKey[index]}`;
  postman.setEnvironmentVariable(FsPlusNameTrue, FsPlusName);
  
  var fsKey = `amount/fs/${language}`;
  var fsText = pm.environment.get(fsKey);
  
  var titleKey = `translations/title/${language}`;
  var titleText = pm.environment.get(titleKey);
  var descriptionKey = `description/translations/${language}`;
  var descriptionText = pm.environment.get(descriptionKey);

  pm.test(`Amount of freespin from title on ${language} language are correct`, () => {
    if (Array.isArray(freespinsKey[index])) {
      pm.expect(titleText).to.satisfy((text) => {
        return text.includes(fsText) || freespinsKey[index].slice(1).some(key => text.includes(key));
      });
    } else {
      pm.expect(titleText).to.include(fsText);
    }
  });

  pm.test(`Amount of freespin from description on ${language} language are correct`, () => {
    if (Array.isArray(freespinsKey[index])) {
      pm.expect(descriptionText).to.satisfy((text) => {
        return text.includes(fsText) || freespinsKey[index].slice(1).some(key => text.includes(key));
      });
    } else {
      pm.expect(descriptionText).to.include(fsText);
    }
  });
});

languages.forEach(language => {
  pm.test(`Name Game on ${language.toUpperCase()} language is correct`, function () {
    const environmentVariable = `description/translations/${language}`;
    pm.expect(pm.environment.get(environmentVariable)).to.include(pm.environment.get("GameName_true"));
  });
});

var variables = ["deposit_fulfillment", "FREE-SPIN-TPL", "range", "max_bet", "max_grant"];
for (var i = 0; i < languages.length; i++) {
    var language = languages[i];
    for (var j = 0; j < variables.length; j++) {
        var variable = variables[j];
        pm.test(`Variable "${variable}" works correctly in "${language}" language`, function () {
            pm.expect(pm.environment.get(`description/translations/${language}`)).to.not.include(pm.environment.get(variable));
        });
    }
}


var freespinsKeyWager = ["x1","1","1","x1","x1","x1","x1","x1","x1","x1","x1","x1",];
var wagerTrue = parseInt(pm.environment.get("Wager_true"));

languages.forEach((language, index) => {
  let wagerEnv = `wager/${language}`;
  let wagerTrueEnv = `${Array.isArray(freespinsKeyWager[index]) ? freespinsKeyWager[index][0] : freespinsKeyWager[index]}`;
  postman.setEnvironmentVariable(wagerEnv, wagerTrueEnv);
  
  let wagerrr = `wager/${language}`;
  let Wagers = pm.environment.get(wagerrr);
  
  let descriptionKey = `description/translations/${language}`;
  let descriptionText = pm.environment.get(descriptionKey);

  pm.test(`Wager from description on ${language} language are correct`, () => {
    if (Array.isArray(freespinsKeyWager[index])) {
      pm.expect(descriptionText).to.satisfy((text) => {
        return text.includes(Wagers) || text.includes(freespinsKeyWager[index][1]);
        
      });
    } else {
      pm.expect(descriptionText).to.include(Wagers);
    }
  });
});


var freespinsKeyValidatetime = ["horas","saat","часа","hours","hours","Stunden","heures","godzin","hours","horas","timmar","時"];
var ValidatetimeTrue = parseInt(pm.environment.get("Bonus_Valid_time_true"));

languages.forEach((language, index) => {
  const time = `validatetime/${language}`;
  let timetrue;

  if (language === "ja") {
    timetrue = `${ValidatetimeTrue}時間`;
  } else {
    timetrue = `${ValidatetimeTrue} ${Array.isArray(freespinsKeyValidatetime[index]) ? freespinsKeyValidatetime[index][0] : freespinsKeyValidatetime[index]}`;
  }
  postman.setEnvironmentVariable(time, timetrue);
  
  const timeKey = `validatetime/${language}`;
  const timeText = pm.environment.get(timeKey);
  const descriptionKey = `description/translations/${language}`;
  const descriptionText = pm.environment.get(descriptionKey);

  pm.test(`Validate time from description on ${language} language are correct`, () => {
    if (Array.isArray(timeKey[index])) {
      pm.expect(descriptionText).to.satisfy((text) => {
        return text.includes(timeText) || text.includes(timeKey[index][1]);
      });
    } else {
      pm.expect(descriptionText).to.include(timeText);
    }
  });
});

pm.test("Max Bet are correct", function () {
  languages.forEach(function (language) {
    const descriptionKey = "description/" + language + "_full_String";
    pm.expect(pm.environment.get(descriptionKey)).to.include(pm.environment.get("Maximum_bet_true"));
  });
});

pm.test("Max Grant are correct", function () {
  languages.forEach(function (language) {
    const descriptionKey = "description/" + language + "_full_String";
    pm.expect(pm.environment.get(descriptionKey)).to.include(pm.environment.get("Maximum_amount"));
  });
});

languages.forEach(function (language) {
    let asdValue = pm.environment.get("description/translationsWithoutData/" + language);
    var qwe = pm.environment.get("AmountFreeSpin_true");
    var NumberWithGameName = pm.environment.get("extractedNumberVariable");

    
    if (asdValue !== null) {
        let foundNumbers = asdValue.match(/\b\d+\b/g);

        let allNumbersLessThanOrEqualTo10 = foundNumbers.every(number => parseInt(number) === parseInt(qwe) || parseInt(number) === 1 || parseInt(number) === 24 || parseInt(number) === parseInt(NumberWithGameName));

        pm.test(`Присутні лишні числа окрім, вейджера, fs і bonus validate time for ${language}`, function () {
            pm.expect(allNumbersLessThanOrEqualTo10).to.be.true;
        });
    }
});

pm.test("maxBet/BO are correct", function () {
pm.expect(pm.environment.get("maxBet/BO")).to.eql((pm.environment.get("maxBet/BO_true")))});

pm.test("maxBetSport/BO are correct", function () {
pm.expect(pm.environment.get("maxBetSport/BO")).to.eql((pm.environment.get("maxBetSport/BO_true")))});

pm.test("maxGrantAmount/BO are correct", function () {
pm.expect(pm.environment.get("maxGrantAmount/BO")).to.eql((pm.environment.get("maxGrantAmount/BO_true")))});

pm.test("Campaign Name in BO are correct", function () {
pm.expect(pm.environment.get("name/BO")).to.include((pm.environment.get("GameName_true")));
pm.expect(pm.environment.get("name/BO")).to.include((pm.environment.get("AmountFreeSpin_true")))});

pm.test("Campagin status in BO are correct", function () {
    var stateBO = pm.environment.get("state/BO");
    var stateBOTrue = pm.environment.get("state/BO_true");
    var stateBO1True = pm.environment.get("state/BO1true");

    pm.expect(stateBO.includes(stateBOTrue) || stateBO.includes(stateBO1True)).to.be.true;
});

pm.test("period in BO are correct", function () {
pm.expect(pm.environment.get("period/BO")).to.include((pm.environment.get("period/BO_true")))});

pm.test("action in BO are correct", function () {
pm.expect(pm.environment.get("action/BO")).to.include((pm.environment.get("action/BO_true")))});

pm.test("targetType in BO are correct", function () {
pm.expect(pm.environment.get("targetType/BO")).to.include((pm.environment.get("targetType/BO_true")))});

pm.test("optInPeriod in BO are correct", function () {
pm.expect(pm.environment.get("optInPeriod/BO")).to.include((pm.environment.get("optInPeriod/BO_true")))});

pm.test("optInPeriodTimeUnit in BO are correct", function () {
pm.expect(pm.environment.get("optInPeriodTimeUnit/BO")).to.include((pm.environment.get("optInPeriodTimeUnit/BO_true")))});

pm.test("campaignPeriod in BO are correct", function () {
pm.expect(pm.environment.get("campaignPeriod/BO")).to.include((pm.environment.get("campaignPeriod/BO_true")))});


pm.test("freeSpinsAmount in BO are correct", function () {
pm.expect(pm.environment.get("freeSpinsAmount/BO")).to.include((pm.environment.get("AmountFreeSpin_true")))});

var Zero = pm.environment.get("freeSpinPrice/BO");
var hasZero = Object.values(Zero).some(value => value === 0);
var EUR = pm.environment.get("freeSpinPrice/BO");
var EURtrue = JSON.parse(EUR);
var EURValue = EURtrue["EUR"];
var EUR1 = pm.environment.get("freeSpinPrice/BO_true").replace(/,/g, ".");

var EURtrue1 = null;
if (EUR1) {
    EURtrue1 = JSON.parse(EUR1);
}

pm.test("freeSpinPrice in BO are correct()", function () {
    if (EURtrue1 !== null) {
        pm.expect(EURValue).to.eql(EURtrue1);
    } else {
        pm.expect(EURtrue1).to.eql(null);
    }
});

pm.test("freeSpinPrice in BO dont have zero value", function () {
    pm.expect(hasZero).to.be.false;
});

pm.test("freeSpinsAmount in BO are correct", function () {
    var percentageBO = pm.environment.get("percentage/BO");
    var percentageBO_true = pm.environment.get("percentage/BO_true");

    if (percentageBO && percentageBO_true) {
        pm.expect(percentageBO).to.include(percentageBO_true);
    } else {
        
        pm.expect(true).to.be.true;
    }
});

pm.test("freeSpinsAmount in BO are correct", function () {
pm.expect(pm.environment.get("percentage/BO")).to.include((pm.environment.get("percentage/BO_true")))});

const brackets = ["bracket(1)", "bracket(2)"];

pm.test("Variables in description are correct", function () {
  brackets.forEach((bracket) => {
    languages.forEach((language1) => {
      const descriptionKey = `description/translations/${language1}`;
      const descriptionValue = pm.environment.get(descriptionKey);

      // Подсчет количества открывающих и закрывающих скобок
      const openingBracketCount = (descriptionValue.match(/{/g) || []).length;
      const closingBracketCount = (descriptionValue.match(/}/g) || []).length;

      pm.expect(openingBracketCount).to.eql(closingBracketCount, `Error in ${descriptionKey} (${language1}): Mismatched brackets`);

      if (openingBracketCount !== closingBracketCount) {
        console.error(`Mismatched brackets in ${descriptionKey} (${language1})`);
      }
    });
  });
});


pm.test("Convert numbers in environment variable to format", function () {
    var numbers = pm.environment.get("numbers");
    var currencies = ["EUR", "PLN", "CAD", "AUD", "SEK", "TRY", "GEL", "BRL", "UZS", "UAH", "PEN", "CZK", "AZN", "CLP", "MXN", "KZT", "CHF", "INR", "JPY", "ARS", "USD", "RUB", "NZD", "NOK", "ZAR"];

    var formattedNumbers = numbers.split("\t").map(function (number, index) {
        var currency = currencies[index];
        return '"' + currency + '":"' + number + currency + '"';
    });

    var result = formattedNumbers.join(",");

    pm.environment.set("convertedNumbersForCMS", result);
});

pm.test("Convert numbers in environment variable to JSON format", function () {
    var numbers = pm.environment.get("numbers");
    var currencies = ["EUR", "PLN", "CAD", "AUD", "SEK", "TRY", "GEL", "BRL", "UZS", "UAH", "PEN", "CZK", "AZN", "CLP", "MXN", "KZT", "CHF", "INR", "JPY", "ARS", "USD", "RUB", "NZD", "NOK", "ZAR"];

    var formattedNumbers = numbers.split("\t").map(function (number, index) {
        var currency = currencies[index];
        return { "min": parseInt(number), "max": null, "currency": currency };
    });

    var result = JSON.stringify(formattedNumbers);

    pm.environment.set("convertedNumbersForBO", result);
});

let names = pm.environment.get("GameName_true");
let withoutSpaces = names.replace(/\s/g, "");

var GameName_true = pm.variables.get("GameName_true");
var modifiedGameName_true = GameName_true.replace(/\s/g, "").replace(/-/g, "").toLowerCase();
var gameNameParts = modifiedGameName_true.split(":");
var gameName = gameNameParts[0];
var gameBO = pm.environment.get("game/BO");
var modifiedGameBO = gameBO.replace(/_/g, "");

pm.test("Game in BO are correct", function () {
    pm.expect(modifiedGameBO.toLowerCase()).to.include(gameName)
});

pm.test("Provider in BO are correct", function () {
    var providerBO = String(pm.environment.get("provider/BO"));
    var providerBOTrue = String(pm.environment.get("provider/BO_true"));

    pm.expect(providerBO.toLowerCase()).to.include(providerBOTrue.toLowerCase());
});



