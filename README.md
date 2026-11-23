# Ex.No:3 Develop a simple application to play and control the audio file in android studio.


## AIM:

To develop a simple application, to play and control the audio file and to perfrom the start,pause and stop opeartion in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as audiofile and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml and create start,pause and stop button.

Step 6: Display message give in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```
Program to play and control the audio file”.
Developed by: SANTHOSH D
Registeration Number 212223220099
```
## PROGRAM
## MainActivity.java
```
package com.example.audiofile;

import android.media.AudioManager;
import android.media.MediaPlayer;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.util.Log;
import android.widget.Button;
import android.widget.SeekBar;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

import java.util.Locale;

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    private MediaPlayer mediaPlayer;
    private SeekBar seekBar;
    private SeekBar volumeSeekBar;
    private TextView tvTime;
    private AudioManager audioManager;
    private final Handler handler = new Handler(Looper.getMainLooper());
    private Runnable updateSeekBarRunnable;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        setVolumeControlStream(AudioManager.STREAM_MUSIC);
        audioManager = (AudioManager) getSystemService(AUDIO_SERVICE);

        Button btnPlay = findViewById(R.id.btnPlay);
        Button btnPause = findViewById(R.id.btnPause);
        Button btnStop = findViewById(R.id.btnStop);
        seekBar = findViewById(R.id.seekBar);
        volumeSeekBar = findViewById(R.id.volumeSeekBar);
        tvTime = findViewById(R.id.tvTime);

        setupVolumeControl();

        btnPlay.setOnClickListener(v -> playAudio());
        btnPause.setOnClickListener(v -> pauseAudio());
        btnStop.setOnClickListener(v -> stopPlayer());

        seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                if (fromUser && mediaPlayer != null) {
                    mediaPlayer.seekTo(progress);
                    updateTimeText(progress, mediaPlayer.getDuration());
                }
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {}

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {}
        });
    }

    private void setupVolumeControl() {
        if (audioManager != null) {
            int maxVol = audioManager.getStreamMaxVolume(AudioManager.STREAM_MUSIC);
            int currentVol = audioManager.getStreamVolume(AudioManager.STREAM_MUSIC);

            int initialVol = (currentVol == 0) ? (maxVol / 2) : currentVol;
            if (currentVol == 0) {
                audioManager.setStreamVolume(AudioManager.STREAM_MUSIC, initialVol, 0);
            }

            volumeSeekBar.setMax(maxVol);
            volumeSeekBar.setProgress(initialVol);

            volumeSeekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
                @Override
                public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                    if (fromUser && audioManager != null) {
                        audioManager.setStreamVolume(AudioManager.STREAM_MUSIC, progress, 0);
                    }
                }

                @Override
                public void onStartTrackingTouch(SeekBar seekBar) {}

                @Override
                public void onStopTrackingTouch(SeekBar seekBar) {}
            });
        }
    }

    private void ensureAudioVolume() {
        if (audioManager != null) {
            int currentVol = audioManager.getStreamVolume(AudioManager.STREAM_MUSIC);
            int maxVol = audioManager.getStreamMaxVolume(AudioManager.STREAM_MUSIC);
            if (currentVol == 0) {
                int defaultVol = Math.max(1, maxVol / 2);
                audioManager.setStreamVolume(AudioManager.STREAM_MUSIC, defaultVol, 0);
                volumeSeekBar.setProgress(defaultVol);
                Toast.makeText(this, "Device audio unmuted", Toast.LENGTH_SHORT).show();
            }
        }
    }

    private void playAudio() {
        ensureAudioVolume();

        if (mediaPlayer == null) {
            try {
                mediaPlayer = MediaPlayer.create(this, R.raw.dude_oorum_blood);

                if (mediaPlayer != null) {
                    mediaPlayer.setVolume(1.0f, 1.0f);
                    setupSeekBar();
                    mediaPlayer.start();
                    startSeekBarUpdate();
                    Toast.makeText(this, "Playing audio", Toast.LENGTH_SHORT).show();

                    mediaPlayer.setOnCompletionListener(mp -> stopPlayer());
                } else {
                    Toast.makeText(this, "Error: Could not load audio file", Toast.LENGTH_LONG).show();
                }
            } catch (Exception e) {
                Log.e(TAG, "Playback failed", e);
                Toast.makeText(this, "Playback failed", Toast.LENGTH_SHORT).show();
            }
        } else {
            if (!mediaPlayer.isPlaying()) {
                mediaPlayer.start();
                startSeekBarUpdate();
                Toast.makeText(this, "Playing", Toast.LENGTH_SHORT).show();
            }
        }
    }

    private void pauseAudio() {
        if (mediaPlayer != null && mediaPlayer.isPlaying()) {
            mediaPlayer.pause();
            stopSeekBarUpdate();
            Toast.makeText(this, "Paused", Toast.LENGTH_SHORT).show();
        }
    }

    private void setupSeekBar() {
        if (mediaPlayer != null) {
            int duration = mediaPlayer.getDuration();
            seekBar.setMax(duration);
            updateTimeText(mediaPlayer.getCurrentPosition(), duration);
        }
    }

    private void startSeekBarUpdate() {
        stopSeekBarUpdate();
        updateSeekBarRunnable = new Runnable() {
            @Override
            public void run() {
                if (mediaPlayer != null && mediaPlayer.isPlaying()) {
                    int currentPos = mediaPlayer.getCurrentPosition();
                    int duration = mediaPlayer.getDuration();
                    seekBar.setProgress(currentPos);
                    updateTimeText(currentPos, duration);
                    handler.postDelayed(this, 200);
                }
            }
        };
        handler.post(updateSeekBarRunnable);
    }

    private void stopSeekBarUpdate() {
        if (updateSeekBarRunnable != null) {
            handler.removeCallbacks(updateSeekBarRunnable);
            updateSeekBarRunnable = null;
        }
    }

    private void updateTimeText(int currentMs, int totalMs) {
        if (tvTime != null) {
            tvTime.setText(String.format(Locale.US, "%s / %s", formatMs(currentMs), formatMs(totalMs)));
        }
    }

    private String formatMs(int ms) {
        int seconds = (ms / 1000) % 60;
        int minutes = (ms / (1000 * 60)) % 60;
        return String.format(Locale.US, "%02d:%02d", minutes, seconds);
    }

    private void stopPlayer() {
        stopSeekBarUpdate();
        if (mediaPlayer != null) {
            try {
                if (mediaPlayer.isPlaying()) {
                    mediaPlayer.stop();
                }
                mediaPlayer.release();
            } catch (Exception ignored) {}
            mediaPlayer = null;
            seekBar.setProgress(0);
            if (tvTime != null) {
                tvTime.setText("00:00 / 00:00");
            }
            Toast.makeText(this, "Stopped", Toast.LENGTH_SHORT).show();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        stopPlayer();
    }
}
```
## activity_main.xml
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="24dp"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Audio Player"
        android:textSize="26sp"
        android:textStyle="bold"
        app:layout_constraintBottom_toTopOf="@+id/tvAudioName"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_chainStyle="packed" />

    <TextView
        android:id="@+id/tvAudioName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:text="dude_oorum_blood.mp3"
        android:textColor="?android:attr/textColorSecondary"
        android:textSize="14sp"
        app:layout_constraintBottom_toTopOf="@+id/seekBar"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/tvTitle" />

    <SeekBar
        android:id="@+id/seekBar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        app:layout_constraintBottom_toTopOf="@+id/tvTime"
        app:layout_constraintTop_toBottomOf="@+id/tvAudioName" />

    <TextView
        android:id="@+id/tvTime"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:text="00:00 / 00:00"
        android:textSize="12sp"
        app:layout_constraintBottom_toTopOf="@+id/volumeContainer"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/seekBar" />

    <LinearLayout
        android:id="@+id/volumeContainer"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:gravity="center_vertical"
        android:orientation="horizontal"
        app:layout_constraintBottom_toTopOf="@+id/controls"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/tvTime">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Volume: "
            android:textSize="14sp" />

        <SeekBar
            android:id="@+id/volumeSeekBar"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />
    </LinearLayout>

    <LinearLayout
        android:id="@+id/controls"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:orientation="horizontal"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/volumeContainer">

        <Button
            android:id="@+id/btnPlay"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginEnd="8dp"
            android:text="Play" />

        <Button
            android:id="@+id/btnPause"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginEnd="8dp"
            android:text="Pause" />

        <Button
            android:id="@+id/btnStop"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Stop" />

    </LinearLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```
## OUTPUT
<img width="1366" height="768" alt="1st pic" src="https://github.com/user-attachments/assets/5e3c06cd-407d-48e9-9ab1-6c1966fc5766" />
<img width="1366" height="768" alt="2nd" src="https://github.com/user-attachments/assets/de4a2ca4-61f2-4d95-911e-b1d11503ff71" />



## RESULT
   Thus a simple application, to play and control the audio file and to perfrom the start,pause and stop opeartion in Android Studio is developed and executed successfully.
