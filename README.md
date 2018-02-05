# Cloudinary + React Dropzone
[James Carlson](https://github.com/james-carlson). If you see something wrong or that can be improved, let me know.  

## Overview
When students working on React projects want to make it possible for users to upload and store images, the `react-dropzone` (https://www.npmjs.com/package/react-dropzone) npm package can be coupled with a free Cloudinary (https://cloudinary.com/) account.  

Cloudinary is a pretty great image management solution. Students can get an account for free and get enough space to do what they need for their projects and even a little beyond. You can also control the image with query paramters (resize, rotate, etc., see [here](https://cloudinary.com/documentation/image_transformations)).  

Overall, the Cloudinary documentation is great and React Dropzone's is pretty good. If students take a look at that, do their best, and then you have this, you should be able to get things up and running.

## Psuedo Code
*Import the Dropzone component from 'react-dropzone' to the target component.
*Place the component in your render function where you'd like it to appear.
*Use the "onDrop" function to set the image to state.
*Also fire a function that hits the cloudinary API with an upload. 
*The response from uploading the image will contain a URL. 
*Save the *URL* to the students database (rather than the image). 
*When wanting to display images that have been uploaded, you stick the URL in the src attribute(s) of an `img` tag.


## Actual example
Here's some code I used on a project to get these two playing nicely. If this isn't clear, I also have this code with things more spelled out below. 

```
import React from 'react';
import axios from 'axios';
import Dropzone from 'react-dropzone';
import { connect } from 'react-redux';
import { newCloudinaryUrl } from '../../ducks/reducer';

const preset = process.env.REACT_APP_CLOUDINARY_UPLOAD_PRESET;
const url = process.env.REACT_APP_CLOUDINARY_UPLOAD_URL;


class PhotoUploader extends React.Component {
    constructor() {
        super()
        this.state = { files: [] }
    }

    onImageDrop(files) {
        console.log(files)
        this.setState({
            files: files
        })
        this.handleImageUpload(files)
    }


    handleImageUpload(files) {
        // Push all the axios request promise into a single array
        const uploaders = files.map(file => {
            // Initial FormData
            const formData = new FormData();
            formData.append("file", file);
            formData.append("tags", this.props.user.user_id.toString());
            formData.append("upload_preset", preset); // Replace the preset name with your own
            formData.append("api_key", 428332871437726); // Replace API key with your own Cloudinary key
            formData.append("timestamp", (Date.now() / 1000) | 0);

            // Make an AJAX upload request using Axios (replace Cloudinary URL below with your own)
            return axios.post(url, formData, {
                headers: { "X-Requested-With": "XMLHttpRequest" },
            }).then(response => {
                const data = response.data;
                console.log(data);
                this.props.newCloudinaryUrl(response.data.secure_url)
            })
        });

        // Once all the files are uploaded 
        axios.all(uploaders).then(() => {
            console.log("DONE")
            // ... perform after upload is successful operation
        });

    }


    render() {
        const dropzoneStyle = {
            width: "50%",
            border: "3px dashed #cdcdcd",
            align: "center",
            margin: "1vw auto",
            padding: "1vw",
            borderRadius: "1vw",
            cursor: "pointer"
        };

        return (
            <div className="imagePreview_main">
                <Dropzone multiple={true} accept="image/*" onDrop={(file) => this.onImageDrop(file)}
                    style={dropzoneStyle}>
                    <div>To upload, click here, or drag an drop and image.</div>
                </Dropzone>
                {/* {
                    this.state.files.map(f => <li key={f.name}>{f.name} - {f.size} bytes</li>)
                } */}
                <section className="imagePreview">
                    {this.props.cloudinaryUrl.length !== 0 ? <div><b>Image Preview:</b></div> : null}
                    <div className="imagesDisplay">
                        {this.props.cloudinaryUrl.map((img, i) => {
                            return (
                                <div key={i} className="preview-container">
                                    {/* <a className="photouploader-preview" target="blank" href={this.props.cloudinaryUrl[i]}> */}
                                        <img className="photouploader-preview" src={this.props.cloudinaryUrl[i]} alt="uploaded documentation" />
                                    {/* </a> */}
                                </div>
                            )
                        })}
                    </div>
                </section>
            </div>
        )
    }
}

function mapStateToProps(state) {
    return state
}

const outputActions = {
    newCloudinaryUrl,
}

export default connect(mapStateToProps, outputActions)(PhotoUploader);
```

## Actual example - annotated
This example assumes an app connected to Redux as well. Note that the numbers are just how you would write all of this from scratch, not necessarily the steps taken when you upload an image, etc. A ful

1. Importage
```
import React from 'react';
import axios from 'axios';
import Dropzone from 'react-dropzone';
import { connect } from 'react-redux';
import { newCloudinaryUrl } from '../../ducks/reducer';
```

2. Registering for Cloudinary will get you an upload preset and an upload URL. These are like API keys and should be stored in `.env`. 
```
const preset = process.env.REACT_APP_CLOUDINARY_UPLOAD_PRESET;
const url = process.env.REACT_APP_CLOUDINARY_UPLOAD_URL;
```

3. Create with some local state to hold uploaded files (maybe this is anti-pattern? Sorry)...
```
class PhotoUploader extends React.Component {
    constructor() {
        super()
        this.state = { files: [] }
    }
```

4. Add a a method (here called `onImageDrop`) that 1) sets the files from the Dropzone upload on state, and that also calls the function that will handle upload to Cloudinary.
```
    onImageDrop(files) {
        console.log(files)
        this.setState({
            files: files
        })
        this.handleImageUpload(files)
    }
```

5. Add a method to handle image upload to Cloudinary (vs. the image upload to local state for this component).
```
    handleImageUpload(files) {

        // There is an option to upload multiple images. (Details below, but basically you add this to the Dropzone component: multiple={true}
        // When there are multiple, they just turn into a nice little array on your local state.
        // Now we need to do an axios request for each one.
        const uploaders = files.map(file => { 

            // Setup initial FormData. Using FormData here...not going to lie, this is from a tutorial somewhere that I would reference if I could remember what it was.
            const formData = new FormData();
            formData.append("file", file);
            formData.append("tags", this.props.user.user_id.toString());
            formData.append("upload_preset", preset); // Replace the preset name with your own, see number 2.
            formData.append("api_key", 111111111111111); // Replace API key (111...) with your own Cloudinary key
            formData.append("timestamp", (Date.now() / 1000) | 0); // If you want a timestamp

            // Make an AJAX upload request using Axios (replace Cloudinary URL below with your own)
            return axios.post(url, formData, {   // url is coming from step 2
                headers: { "X-Requested-With": "XMLHttpRequest" },
            }).then(response => {
                const data = response.data;
                console.log(data);
                this.props.newCloudinaryUrl(response.data.secure_url) // fire an action to send the url to reducer
            })
        });

        // Once all the files are uploaded 
        axios.all(uploaders).then(() => {
            console.log("DONE") // ... perform after upload is successful operation
        });

    }
```
6. Start the render and style the dropzone, if you wish.
```
    render() {
        const dropzoneStyle = {
            width: "50%",
            border: "3px dashed #cdcdcd",
            align: "center",
            margin: "1vw auto",
            padding: "1vw",
            borderRadius: "1vw",
            cursor: "pointer"
        };
```

7. JSX: Do stuff like position your Dropzone properly, allow for multiple (`multiple={true}`), accept all image types (`accept="image/*"`) what function to fire when you "Drop" an image (`onDrop={(file) => this.onImageDrop(file)`) and styling (`style={dropzoneStyle}>`, made in step 6.)
```
        return (
            <div className="imagePreview_main">
                <Dropzone multiple={true} accept="image/*" onDrop={(file) => this.onImageDrop(file)}
                    style={dropzoneStyle}>
                    <div>To upload, click here, or drag an drop and image.</div>
                </Dropzone>
                {/* While you're getting things setup this can help you see whether you're getting images on state properly: 
                {
                    this.state.files.map(f => <li key={f.name}>{f.name} - {f.size} bytes</li>)
                } 
                */}
                <section className="imagePreview">
                    {this.props.cloudinaryUrl.length !== 0 ? <div><b>Image Preview:</b></div> : null}  // Here we make it so users can see what they've uploaded. If they have uploaded anything. And the URL has properly gone through the reducer and the cloudinary URL is available on state.
                    <div className="imagesDisplay">
                        {this.props.cloudinaryUrl.map((img, i) => {
                            return (
                                <div key={i} className="preview-container">
                                    {/* <a className="photouploader-preview" target="blank" href={this.props.cloudinaryUrl[i]}> */}
                                        <img className="photouploader-preview" src={this.props.cloudinaryUrl[i]} alt="uploaded documentation" />
                                    {/* </a> */}
                                </div>
                            )
                        })}
                    </div>
                </section>
            </div>
        )
    }
}
```

8. Wrap up Redux.
```
function mapStateToProps(state) {
    return state
}

const outputActions = {
    newCloudinaryUrl,
}

export default connect(mapStateToProps, outputActions)(PhotoUploader);
```
