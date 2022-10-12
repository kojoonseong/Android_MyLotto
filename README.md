# 안드로이드 로또 뽑는 앱
(Android Studio / Kotlin) Lotto Application

### 시작화면
<img src="https://user-images.githubusercontent.com/93521155/180141952-78a5d60b-edee-40b7-b0d7-8f70219d739c.png" width="540px" height="960px"></img>
<br>
<br>
### 실행화면
"로또번호 생성하기"라는 버튼을 누르게 되면, 로또 번호가 추첨된다.<br>
<img src="https://user-images.githubusercontent.com/93521155/180143041-4b4e5f76-5784-4c40-8b90-62f01b06ba8b.png" width="540px" height="960px"></img><br>
하지만 중복된 번호를 뽑고 싶다면 "중복 번호 허용"에 체크를 한 후 뽑게 되면 중복된 번호가 나온다.<br>
<img src="https://user-images.githubusercontent.com/93521155/180362755-fbf8b0f0-ae0c-4b82-ae2d-94b85eb98270.png" width="540px" height="960px"></img><br>
이 번호를 내가 저장하고 싶다면 "저장하기"를 누른 후 "생성한 로또 리스트 확인하기"에서 저장된 로또 번호를 확인할 수 있다.<br>
<img src="https://user-images.githubusercontent.com/93521155/180363046-aeb1d5d2-467d-44ce-bbdf-05e6d352ef35.png" width="540px" height="960px"></img><br>
<img src="https://user-images.githubusercontent.com/93521155/180363139-bed7e6b2-4572-4fe3-ba85-72cc8c841977.png" width="540px" height="960px"></img><br>
<br><br>
### 코드
##### MainActivity
```kotlin
package com.example.mylotto

import android.content.Context
import android.content.Intent
import android.os.Bundle
import android.util.Log
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.core.content.ContextCompat
import com.example.mylotto.databinding.ActivityMainBinding
import java.util.*

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private var idx = -1
    private var listLotto = arrayOfNulls<String>(30)
    private var lastSaved:String? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        initViews()
        loadPref()
        
        binding.btnNew.setOnClickListener {
            if(binding.cbIsSame.isChecked) {
                getLottoNumber1()
            } else {
                getLottoNumber2()
            }
            binding.btnSave.isEnabled = true
        }

        binding.btnSave.setOnClickListener {
            val lotto = IntArray(6)
            lotto[0] = binding.tvLotto1.text.toString().toInt()
            lotto[1] = binding.tvLotto2.text.toString().toInt()
            lotto[2] = binding.tvLotto3.text.toString().toInt()
            lotto[3] = binding.tvLotto4.text.toString().toInt()
            lotto[4] = binding.tvLotto5.text.toString().toInt()
            lotto[5] = binding.tvLotto6.text.toString().toInt()

            var strLotto = ""
            for(i in lotto.indices) {
                strLotto += lotto[i].toString()
                if(i != lotto.size - 1) strLotto += ","
            }
            if(strLotto == lastSaved) {
                Toast.makeText(this, "이미 저장된 로또입니다.", Toast.LENGTH_SHORT).show()
                idx--
                return@setOnClickListener
            }

            idx++
            if(idx == 30) {
                Toast.makeText(this, "저장공간이 없습니다", Toast.LENGTH_SHORT).show()
                idx--
                return@setOnClickListener
            }
            listLotto[idx] = strLotto
            lastSaved = strLotto
            Toast.makeText(this, "저장완료", Toast.LENGTH_SHORT).show()
            Log.d("디버깅",strLotto)
            binding.btnShowList.isEnabled = true

            savePref()
        }

        binding.btnShowList.setOnClickListener {
            var strListLotto:String = ""
            for(i in 0..idx) {
                strListLotto += listLotto[i]
                if(i != idx) strListLotto += ","
            }
            val intent = Intent(this, ResultActivity::class.java)
            intent.putExtra("strListLotto", strListLotto)
            startActivity(intent)
        }
    }

    private fun savePref() {
        var strListLotto: String = ""
        for(i in 0..idx) {
            strListLotto += listLotto[i]
            if(i != idx) strListLotto += "|"
        }
        val sp = getSharedPreferences(KEY_PREF, Context.MODE_PRIVATE)
        val editer = sp.edit()
        editer.putInt(KEY_INDEX, idx)
        editer.putString(KEY_LASTSAVED, lastSaved)
        editer.putString(KEY_LOTTO, strListLotto)
        editer.apply()
    }

    private fun loadPref() {
        val sp = getSharedPreferences(KEY_PREF, Context.MODE_PRIVATE)
        if(sp.contains(KEY_INDEX)) {
            idx = sp.getInt(KEY_INDEX, -1)
            lastSaved = sp.getString(KEY_LASTSAVED, null)
            val strListLotto: String? = sp.getString(KEY_LOTTO, null)
            val arrListLotto: List<String>? = strListLotto?.split("|")
            for(i in 0..idx) {
                listLotto[i] = arrListLotto?.get(i)
            }
            binding.btnShowList.isEnabled = true
        }
    }

    private fun initViews() {
        binding.btnSave.isEnabled = false
        binding.btnShowList.isEnabled = false
        binding.tvLotto1.setBackgroundResource(R.drawable.circle_white)
        binding.tvLotto2.setBackgroundResource(R.drawable.circle_white)
        binding.tvLotto3.setBackgroundResource(R.drawable.circle_white)
        binding.tvLotto4.setBackgroundResource(R.drawable.circle_white)
        binding.tvLotto5.setBackgroundResource(R.drawable.circle_white)
        binding.tvLotto6.setBackgroundResource(R.drawable.circle_white)
    }

    private fun getLottoNumber1() {
        val rnd = Random()
        val lotto = IntArray(6)
        for(i in 0 until 6){
            lotto[i] = rnd.nextInt(45) + 1
        }
        setTvNumber(lotto[0], binding.tvLotto1)
        setTvNumber(lotto[1], binding.tvLotto2)
        setTvNumber(lotto[2], binding.tvLotto3)
        setTvNumber(lotto[3], binding.tvLotto4)
        setTvNumber(lotto[4], binding.tvLotto5)
        setTvNumber(lotto[5], binding.tvLotto6)
    }

    private fun setTvNumber(i: Int, tv: TextView) {
        tv.text = i.toString()
        when(i){
            in 1..10 -> tv.background = ContextCompat.getDrawable(this, R.drawable.circle_blue)
            in 11..20 -> tv.background = ContextCompat.getDrawable(this, R.drawable.circle_red)
            in 21..30 -> tv.background = ContextCompat.getDrawable(this, R.drawable.circle_purple)
            in 31..40 -> tv.background = ContextCompat.getDrawable(this, R.drawable.circle_mint)
            else -> tv.background = ContextCompat.getDrawable(this, R.drawable.circle_green)
        }
    }

    private fun getLottoNumber2() {
        val rnd = Random()
        var lotto = IntArray(6)
        while(true) {
            var isSame = false
            for(i in lotto.indices) {
                lotto[i] = rnd.nextInt(45) + 1
            }
            for (i in lotto.indices) {
                for (j in lotto.indices) {
                    if(i != j) {
                        if(lotto[i] == lotto[j]) {
                            isSame = true
                        }
                    }
                }
            }
            if(!isSame) {
                break
            }
        }
        setTvNumber(lotto[0], binding.tvLotto1)
        setTvNumber(lotto[1], binding.tvLotto2)
        setTvNumber(lotto[2], binding.tvLotto3)
        setTvNumber(lotto[3], binding.tvLotto4)
        setTvNumber(lotto[4], binding.tvLotto5)
        setTvNumber(lotto[5], binding.tvLotto6)
    }

    companion object {
        private const val KEY_PREF = "lotto_pref"
        private const val KEY_LOTTO = "lotto_list"
        private const val KEY_INDEX = "list_index"
        private const val KEY_LASTSAVED = "last_saved_value"
    }
}
```
##### ResultActivity
```kotlin
package com.example.mylotto

import android.graphics.Color
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.view.Gravity
import android.widget.LinearLayout
import android.widget.TextView
import com.example.mylotto.databinding.ActivityResultBinding

class ResultActivity : AppCompatActivity() {
    private lateinit var binding: ActivityResultBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityResultBinding.inflate(layoutInflater)
        setContentView(binding.root)
        val strListLotto = intent.getStringExtra("strListLotto")
        var arrListLotto: List<String>? = strListLotto?.split(",")
        if(arrListLotto != null) {
            var cnt = 0
            for(s in arrListLotto) {
                cnt++
                if(cnt % 6 == 1) {
                    val tvIndex = TextView(this)
                    tvIndex.text = "\n${(cnt/6+1)}번째"
                    binding.gridLayout.addView(tvIndex)
                }
                binding.gridLayout.addView(getTextViewWithNumber(s))
            }
        }
    }

    private fun getTextViewWithNumber(s: String): TextView {
        val tv = TextView(this)
        tv.text = s
        tv.setTextColor(Color.parseColor("#FFFFFF"))
        tv.gravity = Gravity.CENTER
        when(s.toInt()) {
            in 1..10 -> tv.setBackgroundResource(R.drawable.circle_blue)
            in 11..20 -> tv.setBackgroundResource(R.drawable.circle_red)
            in 21..30 -> tv.setBackgroundResource(R.drawable.circle_purple)
            in 31..40 -> tv.setBackgroundResource(R.drawable.circle_mint)
            else -> tv.setBackgroundResource(R.drawable.circle_green)
        }
        val lp = LinearLayout.LayoutParams(
            LinearLayout.LayoutParams.WRAP_CONTENT,
            LinearLayout.LayoutParams.WRAP_CONTENT
        )
        lp.setMargins(2, 90, 2, 2)
        tv.layoutParams = lp
        return tv
    }
}
```
