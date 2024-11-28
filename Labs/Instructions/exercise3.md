# Exercise 3: Testing the Application: API and Web Components

### Estimated Duration: 30 Minutes

## Lab Scenario

In this exercise, you will validate the functionality of the application by running both the API and web components. You will ensure that each part operates correctly and can seamlessly communicate with one another, confirming that the entire application is functioning as expected by generating the articles from the application.

## Lab Objectives

After you complete this exercise, you will be:

 -  Testing the Application Locally: API and Web.

### Task 1: Testing the Application Locally: API and Web

In this task, you will learn how to test the application locally by running both the API and web parts on your local machine. You will set up the necessary environment, run the components, and verify their functionality.

1. As **Visual Studio Code** is already open, navigate back to your **Git Bash terminal** and navigate to `/src/api` from the **Explorer** menu. 

   ```bash
   cd src/api
   ```

1. Now, as you are inside the `API` directory, run the following command to start your **API** component.

   ```bash
   fastapi dev main.py
   ```

1. Once you run this command, the **API (backend)** will be running on your local system. Please make sure you don't close the terminal or stop the process.

1. Now, to run the **WEB (front end),** you have to open a new tab on the **Terminal**.

1. From the top right corner, click on **v (1)** and select **Git Bash (2)** to open a new tab of the **Git Bash terminal**.

   ![](../media/ex1img2.png)

1. Once you are in the new tab in the **Git Bash** terminal, navigate to the `/src/web` directory.

   ```bash
   cd src/web
   ```

1. Now that you have navigated to the `web` directory, run the following command first to install all the dependencies and set up the environment.

   ```bash
   npm install
   ```

1. Once you have installed all the dependencies and the environment is set up, run the following command to start the frontend service.

   ```bash
   npm run dev
   ```
   
   >By running this command, the frontend will be started on your local system. Make sure you don't close the terminal tab.

1. Now, you will see the URL for the application, which is running locally. To access that, hover over the link and do **CTRL+Click**. It will directly navigate to the application interface.

   ![](../media/ex2img7.png)

1. Once you navigate to your application, you will see an interface similar to this.

   ![](../media/ex2img6.png)

1. On the application pane, click on **Example** to generate an example prompt for the article generation. 

   ![](../media/ex2img5.png)

1. Now, as the prompt is ready, click on **Start Work** to generate the article.

   ![](../media/ex2img4.png)

   >It will take some time to generate the article. Please wait till it is generated.

   ![](../media/ex2img3.png)

1. Now, as the article is generated, you can click on the **Debug** symbol from the bottom right corner to check how the process is going in the backend.

   ![](../media/ex2img2.png)

   >This will open up a new tab from right in which you will be able to trace allthe processes happening in backend.

   ![](../media/ex2img1.png)

1. Now, you can try modifying the example prompt and changing the topic and instruction to play with the application.

## Summary

In this exercise, you have tested the application by running both the API and web components. You verified that each component is functioning properly and that they can communicate with each other seamlessly, ensuring the application operates as expected. This process helped ensure that the entire system was ready for further deployment or development stages.

### You have successfully completed this exercise!!
