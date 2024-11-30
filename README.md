# Audio_Player




MAINACTIVITY




    package com.example.audio_player;
    
    import android.Manifest;
    import android.annotation.SuppressLint;
    import android.content.Intent;
    import android.content.pm.PackageManager;
    import android.database.Cursor;
    import android.net.Uri;
    import android.os.Build;
    import android.os.Bundle;
    import android.os.Environment;
    import android.provider.MediaStore;
    import android.provider.Settings;
    import android.view.View;
    import android.widget.Button;
    import android.widget.TextView;
    import android.widget.Toast;
    
    import androidx.annotation.NonNull;
    import androidx.annotation.Nullable;
    import androidx.appcompat.app.AppCompatActivity;
    import androidx.core.app.ActivityCompat;
    import androidx.core.content.ContextCompat;
    import androidx.recyclerview.widget.LinearLayoutManager;
    import androidx.recyclerview.widget.RecyclerView;
    
    import java.io.File;
    import java.util.ArrayList;

    public class MainActivity extends AppCompatActivity {

          private static final int REQUEST_CODE_READ_EXTERNAL_STORAGE = 100;
          private static final int REQUEST_CODE_MANAGE_EXTERNAL_STORAGE = 101;
      
          Button takePermissionButton;
          Button dontGivePermissionButton;
          RecyclerView recyclerView;
          TextView noMusicTextView;
          ArrayList<Audio_Modal> songsList = new ArrayList<>();
      
          @SuppressLint("MissingInflatedId")
          @Override
          protected void onCreate(Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);
              setContentView(R.layout.activity_main);
      
              takePermissionButton = findViewById(R.id.permission);
              dontGivePermissionButton = findViewById(R.id.dontGivePermission);
              recyclerView = findViewById(R.id.recycler_view);
              noMusicTextView = findViewById(R.id.no_songs_text);
      
              takePermissionButton.setOnClickListener(new View.OnClickListener() {
                  @Override
                  public void onClick(View view) {
                      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                          if (ContextCompat.checkSelfPermission(getApplicationContext(), Manifest.permission.READ_EXTERNAL_STORAGE)
                                  != PackageManager.PERMISSION_GRANTED) {
                              ActivityCompat.requestPermissions(MainActivity.this,
                                      new String[]{Manifest.permission.READ_EXTERNAL_STORAGE}, REQUEST_CODE_READ_EXTERNAL_STORAGE);
                          } else {
                              loadMusicList();
                              hideButtons();
                          }
                      }
                      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
                          if (!Environment.isExternalStorageManager()) {
                              try {
                                  Intent intent = new Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION);
                                  intent.addCategory("android.intent.category.DEFAULT");
                                  intent.setData(Uri.parse(String.format("package:%s", getApplicationContext().getPackageName())));
                                  startActivityForResult(intent, REQUEST_CODE_MANAGE_EXTERNAL_STORAGE);
                              } catch (Exception e) {
                                  Intent intent = new Intent();
                                  intent.setAction(Settings.ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION);
                                  startActivityForResult(intent, REQUEST_CODE_MANAGE_EXTERNAL_STORAGE);
                              }
                          } else {
                              loadMusicList();
                              hideButtons();
                          }
                      }
      
                      // Hide the button after clicking
                      view.setVisibility(View.GONE);
                  }
              });
      
              dontGivePermissionButton.setOnClickListener(new View.OnClickListener() {
                  @Override
                  public void onClick(View view) {
                      // Exit the app
                      finish();
                  }
              });
      
              if (checkPermission()) {
                  loadMusicList();
                  hideButtons();
              } else {
                  requestPermission();
              }
          }
      
          boolean checkPermission() {
              int result = ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.READ_EXTERNAL_STORAGE);
              return result == PackageManager.PERMISSION_GRANTED;
          }
      
          void requestPermission() {
              if (ActivityCompat.shouldShowRequestPermissionRationale(MainActivity.this,
                      Manifest.permission.READ_EXTERNAL_STORAGE)) {
                  Toast.makeText(MainActivity.this, "READ PERMISSION IS REQUIRED, PLEASE ALLOW FROM SETTINGS",
                          Toast.LENGTH_SHORT).show();
              } else {
                  ActivityCompat.requestPermissions(MainActivity.this,
                          new String[]{Manifest.permission.READ_EXTERNAL_STORAGE}, REQUEST_CODE_READ_EXTERNAL_STORAGE);
              }
          }
      
          @Override
          public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
              super.onRequestPermissionsResult(requestCode, permissions, grantResults);
              if (requestCode == REQUEST_CODE_READ_EXTERNAL_STORAGE) {
                  if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                      loadMusicList();
                      hideButtons();
                  } else {
                      Toast.makeText(this, "Permission Denied", Toast.LENGTH_SHORT).show();
                  }
              }
          }
      
          @Override
          protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
              super.onActivityResult(requestCode, resultCode, data);
              if (requestCode == REQUEST_CODE_MANAGE_EXTERNAL_STORAGE) {
                  if (Environment.isExternalStorageManager()) {
                      loadMusicList();
                      hideButtons();
                  } else {
                      Toast.makeText(this, "Permission Denied", Toast.LENGTH_SHORT).show();
                  }
              }
          }
      
          private void loadMusicList() {
              String[] projection = {
                      MediaStore.Audio.Media.TITLE,
                      MediaStore.Audio.Media.DATA,
                      MediaStore.Audio.Media.DURATION
              };
              String selection = MediaStore.Audio.Media.IS_MUSIC + " != 0";
      
              Cursor cursor = getContentResolver().query(MediaStore.Audio.Media.EXTERNAL_CONTENT_URI, projection, selection, null, null);
              if (cursor != null) {
                  while (cursor.moveToNext()) {
                      Audio_Modal songData = new Audio_Modal(cursor.getString(1), cursor.getString(0), cursor.getString(2));
                      if (new File(songData.getPath()).exists())
                          songsList.add(songData);
                  }
                  cursor.close();
              }
      
              if (songsList.isEmpty()) {
                  noMusicTextView.setVisibility(View.VISIBLE);
              } else {
                  recyclerView.setLayoutManager(new LinearLayoutManager(this));
                  recyclerView.setAdapter(new musicListAdapter(songsList, getApplicationContext()));
              }
          }
      
          private void hideButtons() {
              takePermissionButton.setVisibility(View.GONE);
              dontGivePermissionButton.setVisibility(View.GONE);
          }
      
          @Override
          protected void onResume() {
              super.onResume();
              if (recyclerView!=null){
                  recyclerView.setAdapter(new musicListAdapter(songsList,getApplicationContext()));
              }
          }
          
        }
      
