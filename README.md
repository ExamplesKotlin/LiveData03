# LiveData


```
val starLiveData = MutableLiveData<Star>()  //  <<== LiveData
  val emittingLiveData = MutableLiveData<Boolean>()
```

Agregando el Observer:

```
val theObserver = Observer<Star> {
      animateStar(it)
    }

    starViewModel.starLiveData.observe(this, theObserver)

    starViewModel.emittingLiveData.observe(this, Observer { emitting ->
      resetButton.isEnabled = emitting ?: false
      startButton.isEnabled = !resetButton.isEnabled
    })
```

Los archivos completos:

StarViewModel

```
/*
 * Copyright (c) 2018 Razeware LLC
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 *
 * Notwithstanding the foregoing, you may not use, copy, modify, merge, publish,
 * distribute, sublicense, create a derivative work, and/or sell copies of the
 * Software in any work that is designed, intended, or marketed for pedagogical or
 * instructional purposes related to programming, coding, application development,
 * or information technology.  Permission for such use, copying, modification,
 * merger, publication, distribution, sublicensing, creation of derivative works,
 * or sale is expressly withheld.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 * THE SOFTWARE.
 *
 */

package com.raywenderlich.android.starfield.ui

import android.arch.lifecycle.MutableLiveData
import android.arch.lifecycle.ViewModel
import com.raywenderlich.android.starfield.model.Star
import com.raywenderlich.android.starfield.model.Vector
import com.raywenderlich.android.starfield.utils.rand
import java.util.*
import kotlin.concurrent.schedule
import kotlin.math.max

class StarViewModel : ViewModel() {

  private var sizeX = 0.0
  private var sizeY = 0.0
  private var origin: Vector = Vector(sizeX / 2.0, sizeY / 2.0)
  private var starVectorMagnitude = max(sizeX, sizeY)
  private var timer = Timer("Stars", false)

  val starLiveData = MutableLiveData<Star>()  //  <<== LiveData
  val emittingLiveData = MutableLiveData<Boolean>()

  fun startEmittingStars() {
    emittingLiveData.postValue(true)
    timer.schedule(0, 20) {
      val x = rand(0, sizeX.toInt()).toDouble()
      val y = rand(0, sizeY.toInt()).toDouble()
      val starEndPosition = calculateStarEndPosition(x, y)

      val star = Star(x.toInt(), y.toInt(), starEndPosition.x.toInt(), starEndPosition.y.toInt())
      starLiveData.postValue(star)
    }
  }

  fun stopEmittingStars() {
    emittingLiveData.postValue(false)
    timer.cancel()
    timer = Timer("Stars", false)
  }

  fun setupDisplay(sizeX: Double, sizeY: Double) {
    this.sizeX = sizeX
    this.sizeY = sizeY
    origin = Vector(sizeX / 2.0, sizeY / 2.0)
    starVectorMagnitude = max(sizeX, sizeY)
  }

  private fun calculateStarEndPosition(x: Double, y: Double): Vector {
    // Get end position as vector from origin of magnitude starVectorMagnitude
    // and in the same direction as (x, y).
    // If unitVector is null, then just use original position.
    val position = Vector(x, y) - origin
    val unitVector = position.unitVector()
    return if (unitVector != null) {
          unitVector * starVectorMagnitude + origin
        } else {
          position + origin
        }
  }
}
```

StarActivity

```
/*
 * Copyright (c) 2018 Razeware LLC
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in
 * all copies or substantial portions of the Software.
 *
 * Notwithstanding the foregoing, you may not use, copy, modify, merge, publish,
 * distribute, sublicense, create a derivative work, and/or sell copies of the
 * Software in any work that is designed, intended, or marketed for pedagogical or
 * instructional purposes related to programming, coding, application development,
 * or information technology.  Permission for such use, copying, modification,
 * merger, publication, distribution, sublicensing, creation of derivative works,
 * or sale is expressly withheld.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 * THE SOFTWARE.
 *
 */

package com.raywenderlich.android.starfield.ui

import android.animation.Animator
import android.animation.AnimatorListenerAdapter
import android.animation.AnimatorSet
import android.animation.ObjectAnimator
import android.arch.lifecycle.Observer
import android.arch.lifecycle.ViewModelProviders
import android.graphics.Color
import android.os.Bundle
import android.support.v7.app.AppCompatActivity
import android.util.DisplayMetrics
import android.view.View
import android.view.animation.LinearInterpolator
import android.widget.FrameLayout
import com.raywenderlich.android.starfield.R
import com.raywenderlich.android.starfield.model.Star
import com.raywenderlich.android.starfield.utils.rand
import kotlinx.android.synthetic.main.activity_star.*


class StarActivity : AppCompatActivity() {

  private lateinit var starViewModel: StarViewModel

  private val starViews = mutableListOf<View>()

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_star)

    starViewModel = ViewModelProviders.of(this).get(StarViewModel::class.java)

    setupButtons()

    val theObserver = Observer<Star> {
      animateStar(it)
    }

    starViewModel.starLiveData.observe(this, theObserver)

    starViewModel.emittingLiveData.observe(this, Observer { emitting ->
      resetButton.isEnabled = emitting ?: false
      startButton.isEnabled = !resetButton.isEnabled
    })

  }

  private fun setupButtons() {
    startButton.setOnClickListener {
      starViewModel.startEmittingStars()
    }

    resetButton.setOnClickListener {
      starViewModel.stopEmittingStars()
      starViews.forEach { starField.removeView(it) }
      starViews.clear()
    }
  }

  override fun onResume() {
    super.onResume()
    val displayMetrics = DisplayMetrics()
    windowManager.defaultDisplay.getMetrics(displayMetrics)
    starViewModel.setupDisplay(displayMetrics.widthPixels.toDouble(), displayMetrics.heightPixels.toDouble())
  }
  
  private fun animateStar(star: Star?) {
    if (star != null) {
      val starView = createStarView(star)

      val xAnimator = objectAnimatorOfFloat(starView, "x", star.x.toFloat(), star.endX.toFloat())
      val yAnimator = objectAnimatorOfFloat(starView, "y", star.y.toFloat(), star.endY.toFloat())

      starField.addView(starView)
      starViews.add(starView)

      AnimatorSet().apply {
        play(xAnimator).with(yAnimator)

        addListener(object : AnimatorListenerAdapter() {
          override fun onAnimationEnd(animation: Animator?) {
            starField.removeView(starView)
          }
        })

        start()
      }
    }
  }

  private fun createStarView(star: Star): View {
    val starView = View(this)
    val starSize = rand(MIN_SIZE, MAX_SIZE)
    starView.layoutParams = FrameLayout.LayoutParams(starSize, starSize)
    starView.x = star.x.toFloat()
    starView.y = star.y.toFloat()
    starView.setBackgroundColor(Color.parseColor(STAR_COLOR))
    return starView
  }

  private fun objectAnimatorOfFloat(view: View, propertyName: String, startValue: Float, endValue: Float): ObjectAnimator {
    val animator = ObjectAnimator.ofFloat(view, propertyName, startValue, endValue)
    animator.interpolator = LinearInterpolator()
    animator.duration = DURATION
    return animator
  }

  companion object {
    private const val DURATION = 6000L
    private const val STAR_COLOR = "#ffffff"
    private const val MIN_SIZE = 1
    private const val MAX_SIZE = 8
  }
}

```
