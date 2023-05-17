import java.io.File;
import java.io.IOException;
import java.util.Scanner;
import javax.sound.sampled.*;

import java.util.ArrayList;
import java.util.Collections;

public class Main {
  static float prevVol = 0;
  static float currVol = 0;
  static FloatControl fc;
  static boolean mute = false;
  static boolean loop = false;
  static ArrayList<String> songs = new ArrayList<String>();
  static Clip clip;
  static int playlistIndex = 0;

  public static void clearScreen() {
    System.out.print("\033[H\033[2J");
    System.out.flush();
  }

  public static void play(Clip clip) {
    clip.start();
  }

  public static void stop(Clip clip) {
    clip.stop();
  }

  public static void reset(Clip clip) {
    clip.setMicrosecondPosition(0);
  }

  public static void quit(Clip clip) {
    clip.close();
  }

  public static void loop(Clip clip) {
    if (loop == false) {
      clip.loop(Clip.LOOP_CONTINUOUSLY);
      loop = true;
    } else {
      clip.loop(0);
      loop = false;
    }
  }

  public static void volumeUp() {
    currVol += 1.0f;
    if (currVol > fc.getMaximum())
      currVol = fc.getMaximum();
    fc.setValue(currVol);
    System.out.println("Volume: " + (int) currVol);

  }

  public static void volumeDown() {
    currVol -= 1.0f;
    if (currVol < fc.getMinimum())
      currVol = fc.getMinimum();
    fc.setValue(currVol);
    System.out.println("Volume: " + (int) currVol);

  }

  public static void mute() {
    if (mute == false) {
      prevVol = currVol;
      currVol = fc.getMinimum();
      fc.setValue(currVol);
      mute = true;
    } else {
      currVol = prevVol;
      fc.setValue(currVol);
      mute = false;
    }
  }

  public static void print_menu() {
    System.out.println("CLS : CLEAR SCREEN");
    System.out.println("H : HELP");
    System.out.println("L : LOOP/UNLOOP A SONG");
    System.out.println("M : MUTE/UNMUTE");
    System.out.println("P : PLAY");
    System.out.println("PL : DISPLAY CURRENT PLAYLIST");
    System.out.println("PL+ : ADD SONG TO PLAYLIST");
    System.out.println("PL- : REMOVE SONG FROM PLAYLIST");
    System.out.println("Q : QUIT");
    System.out.println("R : RESET");
    System.out.println("S : STOP");
    System.out.println("SH : SHUFFLE PLAYLIST");
    System.out.println("+ : VOLUME UP");
    System.out.println("- : VOLUME DOWN");
  }

  public static void addSong(String song) {
    File file = new File(song + ".wav");
    if (file.exists() && !file.isDirectory()) {
      songs.add(song);
      System.out.println(song + ".wav has been added to the playlist.");
    } else {
      System.out.println("Couldn't find " + song + ".wav");
    }
  }

  public static void removeSong(String song) {
    songs.remove(song);
    File file = new File(song + ".wav");
    if(file.exists())
    System.out.println(song + ".wav has been removed from the playlist.");
    else
    System.out.println("Couldn't find " + song + ".wav");
  }

  public static void displayPlaylist() {
    System.out.println("Current Playlist : ");
    for (int i = 0; i < songs.size(); i++) {
      System.out.println((i + 1) + ". " + songs.get(i) + ".wav");
    }
  }

  public static void main(String[] args) throws UnsupportedAudioFileException, IOException, LineUnavailableException {

    try (Scanner sc = new Scanner(System.in)) {

      System.out.println("CLS : CLEAR SCREEN");
      System.out.println("H : HELP");
      System.out.println("L : LOOP/UNLOOP A SONG");
      System.out.println("M : MUTE/UNMUTE");
      System.out.println("P : PLAY");
      System.out.println("PL : DISPLAY CURRENT PLAYLIST");
      System.out.println("PL+ : ADD SONG TO PLAYLIST");
      System.out.println("PL- : REMOVE SONG FROM PLAYLIST");
      System.out.println("Q : QUIT");
      System.out.println("R : RESET");
      System.out.println("S : STOP");
      System.out.println("SH : SHUFFLE PLAYLIST");
      System.out.println("+ : VOLUME UP");
      System.out.println("- : VOLUME DOWN");

      String response = "";
      songs.add("sample");

      File file = new File(songs.get(playlistIndex) + ".wav");

      AudioInputStream audioStream = AudioSystem.getAudioInputStream(file.getAbsoluteFile());
      clip = AudioSystem.getClip();
      clip.open(audioStream);

      fc = (FloatControl) clip.getControl(FloatControl.Type.MASTER_GAIN);
      prevVol = fc.getValue();
      currVol = fc.getValue();

      while (!response.equals("Q")) {

        if (songs.size() == 0){
          System.out.println("Playlist emplty! Please add a song.");
        }

        System.out.print(">>");

        response = sc.next();
        response = response.toUpperCase();

        switch (response) {
          case ("CLS"):
            clearScreen();
            break;
          case ("P"):
          if (songs.size() == 0){
            songs.add("sample");
            System.out.println("Playlist is empty! Playing a default song.");
          }
            clip.close();
            file = new File(songs.get(playlistIndex) + ".wav");

            audioStream = AudioSystem.getAudioInputStream(file.getAbsoluteFile());
            clip = AudioSystem.getClip();
            clip.open(audioStream);

            fc = (FloatControl) clip.getControl(FloatControl.Type.MASTER_GAIN);
            playlistIndex++;
            if (playlistIndex >= songs.size())
              playlistIndex = 0;
            play(clip);
            break;
          case ("S"):
            stop(clip);
            break;
          case ("R"):
            reset(clip);
            break;
          case ("L"):
            loop(clip);
            break;
          case ("+"):
            volumeUp();
            break;
          case ("-"):
            volumeDown();
            break;
          case ("M"):
            mute();
            break;
          case ("Q"):
            quit(clip);
            break;
          case ("PL+"):
            System.out.println("Enter number of songs you want to add to playlist : ");
            int numberOfSongs = sc.nextInt();
            sc.nextLine();
            for (int i = 0; i < numberOfSongs; i++) {
              System.out.println("Enter the name of song " + (i + 1) + " : ");
              String title = sc.nextLine();
              File nfile = new File(title + ".wav");
              if (nfile.exists())
                addSong(title);
              else
                System.out.println("No match found!");
            }
            break;
          case ("PL-"):
            System.out.println("Enter the name of song you want to remove : ");
            sc.nextLine();
            String title = sc.nextLine();
            removeSong(title);
            break;
          case ("PL"):
            displayPlaylist();
            break;
          case ("SH"):
            Collections.shuffle(songs);
            System.out.println("Playlist has been shuffled.");
            displayPlaylist();
            break;
          case ("H"):
            print_menu();
            break;
          default:
            System.out.println("Invalid response");
        }

      }
      System.out.println("---exited---");
    } catch (Exception e) {
      System.out.println("" + e);
    }
  }
}