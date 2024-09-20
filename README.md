# Step - 1 Install Dependencies
```
npm i @redux/toolkit react-redux  react-router-dom appwrite @tinymce/tinymce-react html-react-parser react-hook-form
```

# Step - 2 Environment Variables in root of the project(Home Directory) File name = .env & .env.sample 
```
REACT_APP_APPWRITE_URL = "" for cra(process.env.\VARIABLENAME)
VITE_APPWRITE_URL="" for vite(import.meta.env.\VARIABLENAME )
VITE_APPWRITE_PROJECT_ID=""
VITE_APPWRITE_DATABASE_ID=""
VITE_APPWRITE_COLLECTION_ID=""
VITE_APPWRITE_BUCKET_ID=""
```

# Step - 3 Make Project->Database->Collection->Bucket & give permissions to edit update etc. as per requirement

# Step - 4 Make folder in src named 'conf'
## in conf make conf.js 
```
const conf = {
    appwriteUrl: String(import.meta.env.VITE_APPWRITE_URL),
    appwriteProjectId: String(import.meta.env.VITE_APPWRITE_PROJECT_ID),
    appwriteDatabaseId: String(import.meta.env.VITE_APPWRITE_DATABASE_ID),
    appwriteCollectionId: String(import.meta.env.VITE_APPWRITE_COLLECTION_ID),
    appwriteBucketId: String(import.meta.env.VITE_APPWRITE_BUCKET_ID),

}

export default conf; 
```

# Step - 5 Make folder in src named 'appwrite'
## In appwrite make auth.js 
### import statements
```
import conf from '../conf/conf'
import {Client , Account , ID} from "appwrite";
```

### Make a class named AuthService
Now accounts will be made only for different instances 
```
export class AuthService {
    client = new Client();
    constructor(){
        this.client
        .setEndpoint(conf.appwriteUrl)
        .setProject(conf.appwriteProjectId);
        this.account = new Account(this.client);
    }
}
```

### Create account function

```
async createAccount({email,password,name}){
        try{
            const userAccount = await this.account.create(ID.unique(),email,password,name);
            if(userAccount){
                // return userAccount; Call another method
                return this.login(email,password);
            }
            else{
                return userAccount;
            }
        } catch(error){
            throw error;
        }
    }
```
### Login & Logout functions

```
async login({email,password}){
        try{
            return await this.account.createEmailSession(email,password);
        } catch(error){
            return error
        }
    }

    async logout({email,password}){
        try{
            await this.account.deleteSessions(email,password);
        } catch(error){
            return error
        }
    }
```
### getCurrentUser function - To check if user is logged in or not

```
async getCurrentUser({email,password}){
        try{
            return await this.account.get(); 
        } catch(error){
            return error
        }
        return null;
    }
```


### Making object & export statements
```
const authService  = new AuthService();
export default authService;

```

## In appwrite make config.js for file upload and other custom queries
### import statements
```
import conf from '../conf.js'
import {Client , Databases, ID, Storage, Query} from "appwrite";
slug is like an id of a single post
```
 
### Make a class named 'Service'
```
export class Service{
    client = new Client();
    databases;
    bucket;
    constructor(){
        this.client
        .setEndpoint(conf.appwriteUrl)
        .setProject(conf.appwriteProjectId);
        this.databases = new Databases(this.client);
    }
}
```

### Create Post Function
```
async createPost({title,slug, content, featuredImage,status, userId})
    {
        try{
            return await this.databases.createDocument(
                conf.appwriteDatabaseId,
                conf.appwriteCollectionId,
                slug,
                {
                    title,
                    content,
                    featuredImage,
                    status,
                    userId,
                }
            )
        }
        catch(error){
            console.log('Appwrite Service :: createPost :: error',error);
        }
    }
```


### Update Post Function
```
async updatePost(slug,{title, content, featuredImage,status}){
        try {
            return await this.databases.updateDocument(
                conf.appwriteDatabaseId,
                conf.appwriteCollectionId,
                slug,
                {
                    title,
                    content,
                    featuredImage,
                    status,
                }
            )
        } catch (error) {
            console.log('Appwrite Service :: updatePost :: error',error);
        }

    }
```
### Delete Post Function
```
async deletePost(slug){
        try {
            await this.databases.deleteDocument(
                conf.appwriteDatabaseId,
                conf.appwriteCollectionId,
                slug,
            )
            return true;
        } catch (error) {
            console.log('Appwrite Service :: deletePost :: error',error);
            return false;
        }

    }
```
### getPost Function -  to search and get a single post
```
async getPost(slug){
        try {
            return await this.databases.getDocument(
                conf.appwriteDatabaseId,
                conf.appwriteCollectionId,
                slug,
            )
        } catch (error) {
            console.log('Appwrite Service :: getPost :: error',error);
        }

    }
```
### getPosts Function -  to get all posts that are active(as all posts can also contain inactive posts)
```
async getPosts(queries = [Query.equal("status","active")]){
        try {
            return await this.databases.listDocuments(
                conf.appwriteDatabaseId,
                conf.appwriteCollectionId,
                queries,
            )
        } catch (error) {
            console.log('Appwrite Service :: getPosts :: error',error);
            return true;
        }

    }
```
- ### After this File upload/delete methods are there

### Upload File function
```
async uploadFile(file){
        try {
            return await this.bucket.createFile(
                conf.appwriteBucketId,
                ID.unique(),
                file,
            )
        } catch (error) {
            console.log();
            return false;
        }
    }
```
### Delete File function
```
async deleteFile(fileId){
        try {
            return await this.bucket.deleteFile(
                conf.appwriteBucketId,
                ID.unique(),
                file,
            )
        } catch (error) {
            console.log();
            return false;
        }
    }
```

### File preview function
```
getFilePreview(fileId){
        return this.bucket.getFilePreview(
            conf.appwriteBucketId,
            fileId,
        )
    }
```

### Making an instance of the class and exporting it
```
const service = new Service();
export default service;
```





# Step - 6 Make folder in src named 'store'
## Make file in store folder named store.js
### import statements
```
import {configureStore } from 'react-redux';
```

### Make store and export
```
const store = configureStore({
    reducers: {

    }
})

export default store;
```

## Make file in store folder named authSlice.js
### import statements
```
import {createSlice} from '@reduxjs/toolkit';
```

### Define initial state
```
const initialState = {
    status: false,
    userData: null
}
```

### Make Slice - Used to track authentication i.e if the user is authenticated or not
```
const authSlice = createSlice({
    name: "auth",
    initialState,
    reducers: {
        login: (state,action) => {
            state.status = true;
            state.userData = action.payload;
        },
        logout: (state) => {
            state.status = false;
            state.userData = null;
        }
    }
})
```


### export each reducer function and reducers as an object also
```
export const {login,logout} = authSlice.actions;

export default authSlice.reducer;
```


# Step - 7 Making folder in src named 'components'
## Make folder Header in components
### Make file Header.jsx and initialise with rfce
## Make folder Footer in components
### Make file Footer.jsx and initialise with rfce

## Make file index.js and import every component and export it
```
import Header from "./Header/Header";
import Footer from "./Footer/Footer";

export{
    Header,
    Footer
}
```

# Step - 8 Updating App.jsx for states and dispatcher
## import statements
```
import { useEffect, useState } from 'react'
import { useDispatch } from "react-redux";
import authService from './appwrite/auth'
import {login,logout} from './store/authSlice'
```

## Add states and dispatcher
```
function App() {
  const [loading , setLoading] = useState(null);
  const dispatch = useDispatch();
}
```

## Also add useEffect s.t whenever a change in state occurs, it re - renders 
```
useEffect(()=>{
    authService.getCurrentUser()
    .then((userData)=> { 
      if(userData){
         dispatch(login({userData}));
        }
      else{
        dispatch(logout());
      }})
      .finally(()=>setLoading(false))

  },[])
```

## 