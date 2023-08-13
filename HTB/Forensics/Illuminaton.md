# Illumination

> A Junior Developer just switched to a new source control platform. Can you find the secret token?

- Let's see what's inside the folder: 
  
  ```bash
  la
  ```
  
  ![](https://user-images.githubusercontent.com/12828790/168432470-3878eccb-a7f6-4c71-9b60-28bdcb463ba1.png)


- Let's check if there are logs of the past:
  
  ```bash
  git log
  ```
  
  <img width="809" alt="Schermata 2022-05-12 alle 16 40 04" src="https://user-images.githubusercontent.com/12828790/168432583-651903ef-9e72-4583-b03a-adc6b32173a5.png">


- Let's find what was the past token, looking for the third commit
  
  > Thanks to contributors, I removed the unique token as it was a security risk. Thanks for reporting responsibly!
  
  ```bash
  git show 47241a47f62ada864ec74bd6dedc4d33f4374699
  ```
  
  <img width="849" alt="Schermata 2022-05-12 alle 16 52 43" src="https://user-images.githubusercontent.com/12828790/168432632-f2ba78dc-4d2b-41d8-ae4e-2d6556b24193.png">
 

- Token Found:
  
  > SFRCe3YzcnNpMG5fYzBudHIwbF9hbV9JX3JpZ2h0P30=
  
  The Token is in base 64, so:
  
  ```bash
  echo SFRCe3YzcnNpMG5fYzBudHIwbF9hbV9JX3JpZ2h0P30 | base64 -d
  ```
  
  <img width="448" alt="Schermata 2022-05-12 alle 16 55 58" src="https://user-images.githubusercontent.com/12828790/168432673-1ee8a8bc-05ec-4ecc-83ee-8a47d77ea874.png">

