> yes , this  is the Great milestone , so i need to do , files upload to s3 as a new feature in API , at the moment the media controller is responsible for that , but we have to refine this according to our new steps , so can you plz trace out the logic starts from media controller

> its fine , but we have to consider the logic of files upload

Requirement:
We have to organize the files under every activities , such that levels --> lesssons --> [photos , videos , audios] , this how the structure looks in s3 , then we have access to preview from s3 , so shall we first setup these things in aws console , after that we can write appropriate apis to solve this 

> now plz guide me to set up appsettings.json related aws s3 configuratiions

> great , shall we now refine the media controller logic to handle single file upload and multiple file upload , with our directory structure as we discussed

> great , we have logic for upload , then how can we view images , vedios , audio  

---

Excellent. Let's integrate the `MediaService` and a `MediaController` into your new Clean Architecture. This is a perfect example of how the architecture makes it easy to add new features.

You are correct that the old `MediaService` needs to be refactored to fit our new patterns (like using the Options Pattern instead of `IConfiguration`).

Here is the complete, step-by-step guide to implement the S3 file upload feature, starting from the controller and working our way down.

---

### **The User Flow**

1.  A client (like your React admin panel) sends a file via a `POST` request to `api/media/upload`.
2.  The `MediaController` receives this request.
3.  It passes the file data to the `IMediaService`.
4.  The `MediaService` uses the `IAmazonS3` client to upload the file to your S3 bucket.
5.  It returns the public URL of the uploaded file.

---

### **Step 1: Refactor the `MediaService` (Application Layer)**

This is the core of the refactoring. We will make it clean, testable, and free of infrastructure dependencies.

**File: `src/ICEDT_TamilApp.Application/Services/Implementation/MediaService.cs`**
*(This refactors the code we discussed previously to fit your current project)*
```csharp
using Amazon.S3;
using Amazon.S3.Model;
using ICEDT_TamilApp.Application.Common; // For AwsSettings
using ICEDT_TamilApp.Application.DTOs.Request;
using ICEDT_TamilApp.Application.DTOs.Response;
using ICEDT_TamilApp.Application.Exceptions;
using ICEDT_TamilApp.Application.Services.Interfaces;
using Microsoft.AspNetCore.Http; // We need this for IFormFile
using Microsoft.Extensions.Options;
using System.Threading.Tasks;

namespace ICEDT_TamilApp.Application.Services.Implementation
{
    public class MediaService : IMediaService
    {
        private readonly IAmazonS3 _s3Client;
        private readonly AwsSettings _awsSettings;

        // The constructor is clean and only asks for what it needs.
        public MediaService(IAmazonS3 s3Client, IOptions<AwsSettings> awsOptions)
        {
            _s3Client = s3Client;
            _awsSettings = awsOptions.Value; // Get the settings from IOptions
        }

        public async Task<MediaUploadResponseDto> UploadFileAsync(IFormFile file, string folder)
        {
            if (file == null || file.Length == 0)
                throw new BadRequestException("File is empty or null.");

            // You can add more validation here (file type, size, etc.)

            // Generate a unique key for the file to prevent name collisions
            var key = $"{folder}/{Guid.NewGuid()}_{Path.GetFileName(file.FileName)}";

            try
            {
                var putRequest = new PutObjectRequest
                {
                    BucketName = _awsSettings.BucketName,
                    Key = key,
                    InputStream = file.OpenReadStream(),
                    ContentType = file.ContentType,
                    // You can add metadata if needed
                    // Metadata = { ["x-amz-meta-original-filename"] = file.FileName }
                };

                await _s3Client.PutObjectAsync(putRequest);
            }
            catch (AmazonS3Exception ex)
            {
                // Log the exception details
                throw new Exception($"Error uploading to S3: {ex.Message}", ex);
            }

            var url = GetPublicUrl(key);

            return new MediaUploadResponseDto
            {
                Url = url,
                Key = key,
                FileName = file.FileName
            };
        }

        private string GetPublicUrl(string key)
        {
            if (string.IsNullOrEmpty(key)) return string.Empty;
            // Note: For eu-north-1 and some other regions, the format might not have the region.
            // Check your S3 object URL format. The most common is below.
            return $"https://{_awsSettings.BucketName}.s3.{_awsSettings.Region}.amazonaws.com/{key}";
        }

        // You can implement DeleteAsync and other methods here later...
    }
}
```
*Note: You will need to create the corresponding `IMediaService` interface and the DTOs (`MediaUploadResponseDto`, `AwsSettings`) if they don't exist yet.*

---

### **Step 2: Create the `MediaController` (Web Layer)**

This controller will be the public endpoint for file uploads.

**File: `src/ICEDT_TamilApp.Web/Controllers/MediaController.cs`**
```csharp
using ICEDT_TamilApp.Application.Services.Interfaces;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using System.ComponentModel.DataAnnotations;
using System.Threading.Tasks;

namespace ICEDT_TamilApp.Web.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class MediaController : ControllerBase
    {
        private readonly IMediaService _mediaService;

        public MediaController(IMediaService mediaService)
        {
            _mediaService = mediaService;
        }

        /// <summary>
        /// Uploads a file to a specified folder in S3.
        /// </summary>
        /// <param name="file">The file to upload.</param>
        /// <param name="folder">The target folder (e.g., 'images', 'audio').</param>
        /// <returns>The public URL and key of the uploaded file.</returns>
        [HttpPost("upload")]
        public async Task<IActionResult> UploadFile(
            [Required] IFormFile file,
            [FromForm, Required] string folder)
        {
            // The [ApiController] attribute automatically handles ModelState validation.
            // If 'file' or 'folder' is missing, it will return a 400 Bad Request.

            var result = await _mediaService.UploadFileAsync(file, folder);

            return Ok(result);
        }
    }
}
```
**Key points about this controller:**
*   It uses `IFormFile` to accept the file upload. This is the standard way in ASP.NET Core.
*   It accepts the `folder` parameter from the form data (`[FromForm]`). This is important because file uploads use `multipart/form-data` content type, not JSON.
*   It's lean and delegates all the actual work to the `_mediaService`.

---

### **Step 3: Configure DI and Settings (`Program.cs` and `appsettings.json`)**

This is the final step to wire everything up.

**1. Update `appsettings.json`**
Make sure you have an `AWS` section with your bucket details.

```json
{
  // ...
  "AWS": {
    "BucketName": "your-deployment-bucket-name", // Or a different bucket for media
    "Region": "eu-north-1", // Your AWS region
    // For local development, you might have keys here.
    // In production on EC2, these should be removed and the IAM Role will be used.
    "AccessKey": "YOUR_LOCAL_DEV_ACCESS_KEY",
    "SecretKey": "YOUR_LOCAL_DEV_SECRET_KEY"
  }
}
```

**2. Update `ApplicationServiceExtensions.cs`**
Register your new service.

```csharp
// In: src/ICEDT_TamilApp.Application/ApplicationServiceExtensions.cs
public static IServiceCollection AddApplicationServices(this IServiceCollection services)
{
    // ... (existing services)
    services.AddScoped<IMediaService, MediaService>();
    
    return services;
}
```

**3. Update `Program.cs` in the `Web` project**
Register the AWS settings and the `IAmazonS3` client.

```csharp
// In: src/ICEDT_TamilApp.Web/Program.cs

// Add these using statements at the top
using Amazon.S3;
using ICEDT_TamilApp.Application.Common; // For AwsSettings

// ...

// *** Configure AWS Services and Settings for DI ***

// 1. Bind the AwsSettings class using the Options Pattern
builder.Services.Configure<AwsSettings>(builder.Configuration.GetSection(AwsSettings.SectionName));

// 2. Add the default AWS options from the configuration (reads the "AWS" section)
builder.Services.AddDefaultAWSOptions(builder.Configuration.GetAWSOptions());

// 3. Register the IAmazonS3 client. 
// The SDK will automatically use the credentials and region from the configuration.
builder.Services.AddAWSService<IAmazonS3>();

// ... (rest of Program.cs)
```
*Remember to install the necessary AWS NuGet packages in your **Web** project: `AWSSDK.S3` and `AWSSDK.Extensions.NETCore.Setup`.*

### **How to Test**

1.  **Run your API.**
2.  Navigate to your Swagger UI (`/swagger`).
3.  Find the new `POST /api/media/upload` endpoint.
4.  Click "Try it out".
5.  You will see two fields:
    *   **file:** A file upload button. Click it and select an image or audio file from your computer.
    *   **folder:** A text box. Type a folder name like `images`.
6.  Click "Execute".

If everything is configured correctly, you will get a `200 OK` response with a JSON body containing the `url` and `key` of the file you just uploaded to your S3 bucket.

---

That is an excellent and much more robust plan. Structuring your S3 bucket like a file system based on your curriculum hierarchy is the perfect way to keep your media assets organized and manageable in the long term.

You are absolutely right. The best workflow is:
1.  **Set up the AWS environment first:** Configure the S3 bucket and the necessary permissions.
2.  **Then, write the API code:** Build the API to work with this pre-defined structure.

Let's walk through the AWS Console setup first.

---

### **Part 1: Setting Up the S3 Bucket and Permissions**

We will create a new S3 bucket (or use an existing one) and configure it to allow public read access, which is necessary for the app to display the images and play the audio/video files.

#### **Step 1: Create or Select Your S3 Bucket**

1.  Navigate to the **S3** service in the AWS Management Console.
2.  If you don't already have a bucket for media, click **"Create bucket"**.
    *   **Bucket name:** Choose a globally unique name, for example: `icedt-tamilapp-media`.
    *   **AWS Region:** Choose the region closest to your users (e.g., `eu-north-1`).
    *   **Block Public Access settings for this bucket:** This is the most important step. You need to **uncheck** the "Block *all* public access" box. This does **not** make your bucket public yet; it just gives you the *ability* to make it public later via a policy. Acknowledge the warning that this might result in the bucket becoming public.
    *   Leave other settings as default and click **"Create bucket"**.

#### **Step 2: Create the Folder Structure in S3**

S3 doesn't technically have "folders," but it simulates them using prefixes. We can create these upfront for clarity.

1.  Click on the name of your new bucket (`icedt-tamilapp-media`).
2.  Click the **"Create folder"** button.
3.  **Folder name:** `levels`. Click **"Create folder"**.
4.  You've created the top-level folder. You don't need to create the sub-folders (like `/levels/1/lessons/3/images/`) manually. Your API will create these automatically when it uploads the first file to that path.

#### **Step 3: Create a Bucket Policy for Public Read Access**

This is the crucial step that allows your Flutter app and React admin panel to display the media files. This policy will state that anyone (`*`) on the internet is allowed to perform the `s3:GetObject` action (i.e., read/download a file) from this bucket.

1.  Inside your bucket, click on the **"Permissions"** tab.
2.  Scroll down to the **"Bucket policy"** section and click **"Edit"**.
3.  Paste the following JSON policy into the editor.

    **IMPORTANT:** You must replace `YOUR-BUCKET-NAME` with your actual bucket name (e.g., `icedt-tamilapp-media`).

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
            }
        ]
    }
    ```
4.  Click **"Save changes"**. You should now see a red "Publicly accessible" tag next to your bucket's name. This is expected and correct for this use case.

#### **Step 4: Configure CORS (Cross-Origin Resource Sharing)**

This is another critical step. By default, a web browser (running your React admin panel) is not allowed to make requests to a domain different from its own (e.g., from your app's domain to your S3 bucket's domain). CORS tells S3 to allow these requests.

1.  Still in the **"Permissions"** tab of your bucket, scroll down to the **"Cross-origin resource sharing (CORS)"** section and click **"Edit"**.
2.  Paste the following JSON configuration. This is a secure but flexible policy for development and production.

    ```json
    [
        {
            "AllowedHeaders": [
                "*"
            ],
            "AllowedMethods": [
                "GET",
                "HEAD"
            ],
            "AllowedOrigins": [
                "http://localhost:3000",
                "https://your-production-admin-domain.com"
            ],
            "ExposeHeaders": []
        }
    ]
    ```
    *   **`AllowedHeaders`:** `*` allows all headers.
    *   **`AllowedMethods`:** We only need `GET` and `HEAD` for public read access.
    *   **`AllowedOrigins`:** This is the most important part.
        *   `http://localhost:3000`: Allows your local React development server to fetch files.
        *   `https://your-production-admin-domain.com`: **Replace this** with the actual domain name of your deployed admin panel. You can also add your Flutter app's domain if it's a web app, or just `*` if you want to allow any origin (less secure, but sometimes necessary).

3.  Click **"Save changes"**.

### **Summary of AWS Console Setup**

Your AWS environment is now perfectly configured:
1.  You have an S3 bucket ready to store your media.
2.  The bucket is configured to allow anyone to **view** the files, which is necessary for the app to function.
3.  The bucket is configured with CORS so your **React admin panel** can display the previews.
4.  Your backend API will still need IAM credentials to **upload** (write) files, which keeps the upload process secure.

You are now ready to write the API code that will interact with this setup.

---

Excellent. Now that the AWS S3 bucket is configured, let's set up your ASP.NET Core application to connect to it securely.

We will configure your `appsettings.json` and `appsettings.Production.json` files. This approach allows you to use secure, long-term credentials for local development while relying on the more secure IAM Role when the application is deployed on the EC2 server.

---

### **Step 1: The `AwsSettings` Class**

First, let's ensure your strongly-typed configuration class in the `Application` layer is complete.

**File: `src/ICEDT_TamilApp.Application/Common/AwsSettings.cs`**
```csharp
namespace ICEDT_TamilApp.Application.Common
{
    public class AwsSettings
    {
        public const string SectionName = "AWS";
        
        // This is needed for constructing the public URL
        public string Region { get; set; } = string.Empty;

        // This is the bucket where media files will be stored
        public string MediaBucketName { get; set; } = string.Empty;

        // --- For Local Development ONLY ---
        // In production, these should be null/empty, and the IAM Role will be used.
        public string? AccessKey { get; set; }
        public string? SecretKey { get; set; }
    }
}
```
*Note: I've renamed `BucketName` to `MediaBucketName` for clarity, in case you have other buckets for other purposes (like deployments) in the future.*

---

### **Step 2: Configure `appsettings.Development.json` (for Your Local Machine)**

This file is used when you run the project locally from VS Code or your terminal. It will contain your IAM user's access keys to allow your local machine to upload files to S3.

**File: `src/ICEDT_TamilApp.Web/appsettings.Development.json`**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=../ICEDT_TamilApp.Infrastructure/Data/tamilapp.db"
  },
  "AWS": {
    "Region": "eu-north-1", // Your AWS Region (e.g., eu-north-1)
    "MediaBucketName": "icedt-tamilapp-media", // Your S3 bucket name
    "AccessKey": "YOUR_IAM_USER_ACCESS_KEY_ID", // <-- PASTE YOUR KEY HERE
    "SecretKey": "YOUR_IAM_USER_SECRET_ACCESS_KEY" // <-- PASTE YOUR SECRET HERE
  }
}
```
**IMPORTANT SECURITY NOTE:** The `appsettings.Development.json` file, which now contains your secret keys, should **NEVER** be committed to a public GitHub repository. Add it to your `.gitignore` file to prevent this.

*   **Add this line to your `.gitignore` file at the root of your repository:**
    ```
    **/appsettings.Development.json
    ```

---

### **Step 3: Configure `appsettings.Production.json` (for the EC2 Server)**

This file will be used when your application runs on the EC2 server. Notice that it **does not contain any access keys**. When the AWS SDK doesn't find keys in the configuration, it will automatically and securely get its credentials from the IAM Role that we attached to the EC2 instance. This is the best practice for production.

**File: `src/ICEDT_TamilApp.Web/appsettings.Production.json`**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=/var/www/database/tamilapp.db"
  },
  "AWS": {
    "Region": "eu-north-1", // Your AWS Region
    "MediaBucketName": "icedt-tamilapp-media" // Your S3 bucket name
    // --- NO ACCESS KEYS HERE ---
  }
}
```

### **Step 4: Update DI Registration in `Program.cs`**

The AWS SDK for .NET is very smart. The existing DI registration code we have is already designed to handle both scenarios (local keys and production IAM roles) perfectly. We just need to ensure it's there.

**File: `src/ICEDT_TamilApp.Web/Program.cs`**
```csharp
// ... (usings)
using Amazon.S3;
using ICEDT_TamilApp.Application.Common;

var builder = WebApplication.CreateBuilder(args);

// ...

// *** THIS SECTION HANDLES BOTH LOCAL AND PRODUCTION CONFIG ***

// 1. Bind the AwsSettings class using the Options Pattern
builder.Services.Configure<AwsSettings>(builder.Configuration.GetSection(AwsSettings.SectionName));

// 2. Add the default AWS options. This reads the "AWS" section.
//    On your local machine, it will find the AccessKey and SecretKey.
//    On the EC2 server, it will NOT find keys and will fall back to using the attached IAM Role.
builder.Services.AddDefaultAWSOptions(builder.Configuration.GetAWSOptions());

// 3. Register the IAmazonS3 client.
builder.Services.AddAWSService<IAmazonS3>();

// ... (rest of your Program.cs)
```

### **Summary of the Configuration**

Your application is now perfectly configured for both development and production environments:

*   **During Local Development:**
    *   The app runs in `Development` mode.
    *   It reads `appsettings.Development.json`.
    *   The `AddDefaultAWSOptions` finds the `AccessKey` and `SecretKey` and configures the S3 client to use them.
    *   You can upload files to S3 directly from your local machine.

*   **On the Deployed EC2 Server:**
    *   The service runs in `Production` (or `Staging`) mode.
    *   It reads `appsettings.Production.json`.
    *   The `AddDefaultAWSOptions` does **not** find any keys.
    *   The SDK automatically detects it's running on an EC2 instance and securely retrieves temporary credentials from the **IAM Role** we attached to the server.
    *   The application can upload files to S3 without any hardcoded secrets.

This setup is secure, flexible, and follows AWS best practices. You are now ready to write the API code that uses this configuration.

---

Excellent. Now that the AWS configuration is solid, let's build the API logic in the `MediaController` and `MediaService` to handle uploads according to your specified S3 directory structure: `levels/{levelId}/lessons/{lessonId}/{mediaType}/`.

We will implement two distinct endpoints:
1.  **Single File Upload:** For uploading a single image or audio file.
2.  **Multiple File Upload:** For uploading a batch of files at once.

---

### **Step 1: Refine the `MediaService` (Application Layer)**

This service will contain the core logic for constructing the S3 key (the "folder path") and performing the upload.

**File: `src/ICEDT_TamilApp.Application/Services/Interfaces/IMediaService.cs` (Updated)**
```csharp
using ICEDT_TamilApp.Application.DTOs.Response;
using Microsoft.AspNetCore.Http;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace ICEDT_TamilApp.Application.Services.Interfaces
{
    public interface IMediaService
    {
        Task<MediaUploadResponseDto> UploadSingleFileAsync(IFormFile file, int levelId, int lessonId, string mediaType);
        Task<List<MediaUploadResponseDto>> UploadMultipleFilesAsync(List<IFormFile> files, int levelId, int lessonId, string mediaType);
    }
}
```

**File: `src/ICEDT_TamilApp.Application/Services/Implementation/MediaService.cs` (Refactored)**
```csharp
using Amazon.S3;
using Amazon.S3.Model;
using ICEDT_TamilApp.Application.Common;
using ICEDT_TamilApp.Application.DTOs.Response;
using ICEDT_TamilApp.Application.Exceptions;
using ICEDT_TamilApp.Application.Services.Interfaces;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Options;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading.Tasks;

namespace ICEDT_TamilApp.Application.Services.Implementation
{
    public class MediaService : IMediaService
    {
        private readonly IAmazonS3 _s3Client;
        private readonly AwsSettings _awsSettings;

        public MediaService(IAmazonS3 s3Client, IOptions<AwsSettings> awsOptions)
        {
            _s3Client = s3Client;
            _awsSettings = awsOptions.Value;
        }

        public async Task<MediaUploadResponseDto> UploadSingleFileAsync(IFormFile file, int levelId, int lessonId, string mediaType)
        {
            if (file == null || file.Length == 0)
                throw new BadRequestException("File is empty or null.");

            // Construct the hierarchical S3 key
            var key = GenerateS3Key(levelId, lessonId, mediaType, file.FileName);

            // Use a shared private method to perform the actual upload
            var url = await UploadToS3Async(file, key);

            return new MediaUploadResponseDto
            {
                Url = url,
                Key = key,
                FileName = file.FileName
            };
        }

        public async Task<List<MediaUploadResponseDto>> UploadMultipleFilesAsync(List<IFormFile> files, int levelId, int lessonId, string mediaType)
        {
            if (files == null || !files.Any())
                throw new BadRequestException("No files provided for upload.");

            var uploadTasks = files.Select(async file =>
            {
                var key = GenerateS3Key(levelId, lessonId, mediaType, file.FileName);
                var url = await UploadToS3Async(file, key);
                return new MediaUploadResponseDto
                {
                    Url = url,
                    Key = key,
                    FileName = file.FileName
                };
            });

            var results = await Task.WhenAll(uploadTasks);
            return results.ToList();
        }

        // --- Private Helper Methods ---

        private string GenerateS3Key(int levelId, int lessonId, string mediaType, string fileName)
        {
            // Sanitize mediaType to prevent path traversal issues (e.g., "images", "audio")
            var sanitizedMediaType = mediaType.ToLowerInvariant();
            if (sanitizedMediaType != "images" && sanitizedMediaType != "audio" && sanitizedMediaType != "videos")
            {
                throw new BadRequestException("Invalid media type. Must be 'images', 'audio', or 'videos'.");
            }
            
            // Generate a unique file name to avoid overwrites
            var uniqueFileName = $"{Guid.NewGuid()}_{Path.GetFileName(fileName)}";

            return $"levels/{levelId}/lessons/{lessonId}/{sanitizedMediaType}/{uniqueFileName}";
        }

        private async Task<string> UploadToS3Async(IFormFile file, string key)
        {
            try
            {
                var putRequest = new PutObjectRequest
                {
                    BucketName = _awsSettings.MediaBucketName,
                    Key = key,
                    InputStream = file.OpenReadStream(),
                    ContentType = file.ContentType
                };

                await _s3Client.PutObjectAsync(putRequest);

                return $"https://{_awsSettings.MediaBucketName}.s3.{_awsSettings.Region}.amazonaws.com/{key}";
            }
            catch (AmazonS3Exception ex)
            {
                // In a real app, log this exception
                throw new Exception($"Error uploading file '{file.FileName}' to S3: {ex.Message}", ex);
            }
        }
    }
}
```

---

### **Step 2: Refine the `MediaController` (Web Layer)**

Now we'll create the two specific endpoints that the admin panel will call.

**File: `src/ICEDT_TamilApp.Web/Controllers/MediaController.cs` (Refactored)**
```csharp
using ICEDT_TamilApp.Application.Services.Interfaces;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Threading.Tasks;

namespace ICEDT_TamilApp.Web.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class MediaController : ControllerBase
    {
        private readonly IMediaService _mediaService;

        public MediaController(IMediaService mediaService)
        {
            _mediaService = mediaService;
        }

        /// <summary>
        /// Uploads a single file to a structured path in S3 based on level and lesson.
        /// </summary>
        [HttpPost("upload-single")]
        [ProducesResponseType(typeof(MediaUploadResponseDto), 200)]
        [ProducesResponseType(400)]
        public async Task<IActionResult> UploadSingleFile(
            [FromForm, Required] IFormFile file,
            [FromForm, Required] int levelId,
            [FromForm, Required] int lessonId,
            [FromForm, Required] string mediaType)
        {
            var result = await _mediaService.UploadSingleFileAsync(file, levelId, lessonId, mediaType);
            return Ok(result);
        }

        /// <summary>
        /// Uploads multiple files to a structured path in S3 based on level and lesson.
        /// </summary>
        [HttpPost("upload-multiple")]
        [ProducesResponseType(typeof(List<MediaUploadResponseDto>), 200)]
        [ProducesResponseType(400)]
        public async Task<IActionResult> UploadMultipleFiles(
            [FromForm, Required] List<IFormFile> files,
            [FromForm, Required] int levelId,
            [FromForm, Required] int lessonId,
            [FromForm, Required] string mediaType)
        {
            var results = await _mediaService.UploadMultipleFilesAsync(files, levelId, lessonId, mediaType);
            return Ok(results);
        }
    }
}
```

### **Summary of the Implementation**

1.  **Hierarchical S3 Keys:** The `GenerateS3Key` private method is now the single source of truth for creating your desired folder structure: `levels/{levelId}/lessons/{lessonId}/{mediaType}/{unique_filename}`. This keeps your S3 bucket perfectly organized.
2.  **Unique File Names:** Using `Guid.NewGuid()` in the file name is a crucial step. It prevents two users from uploading a file with the same name (e.g., `image.png`) and overwriting each other's content.
3.  **Two Distinct Endpoints:** You now have clear, separate endpoints for single and multiple file uploads. Your React admin panel can call the appropriate one. The `upload-multiple` endpoint is particularly powerful, as it processes all files in parallel using `Task.WhenAll` for maximum efficiency.
4.  **Security/Validation:** The service includes basic validation to ensure the `mediaType` is one of the expected values ("images", "audio", "videos"), preventing users from creating arbitrary folders in your S3 bucket.
5.  **Clean Architecture:** The controller remains extremely simple. All the complex logic of key generation, parallel processing, and interacting with the AWS SDK is correctly encapsulated within the `MediaService`.

You are now fully equipped to build a file upload feature in your React admin panel that will automatically organize all media assets in a clean, hierarchical structure within your S3 bucket.

---

That's the perfect follow-up question. Now that the files are neatly organized in S3, we need a way for your admin panel to **browse and select** them.

The best approach is to build a new API endpoint that can list the contents of a specific folder in your S3 bucket. Your React admin panel can then call this endpoint to populate a "Media Browser" or "File Picker" modal.

Let's design and build this feature.

---

### **Step 1: Update the `MediaService` (Application Layer)**

We need to add a new method that can list objects in S3 based on the hierarchical path we created.

**File: `src/ICEDT_TamilApp.Application/Services/Interfaces/IMediaService.cs` (Add this new method)**
```csharp
// ... (existing upload methods)
Task<List<MediaFileDto>> ListFilesAsync(int levelId, int lessonId, string mediaType);
```

**Create a new DTO for the response:**
**File: `src/ICEDT_TamilApp.Application/DTOs/Response/MediaFileDto.cs`**
```csharp
namespace ICEDT_TamilApp.Application.DTOs.Response
{
    public class MediaFileDto
    {
        public string Key { get; set; } = string.Empty; // The full S3 key (path)
        public string Url { get; set; } = string.Empty; // The full public URL
        public string FileName { get; set; } = string.Empty; // The original file name
        public long Size { get; set; }
        public DateTime LastModified { get; set; }
    }
}
```

**Now, implement the logic in the service:**
**File: `src/ICEDT_TamilApp.Application/Services/Implementation/MediaService.cs` (Add this new method)**
```csharp
// ... (existing constructor and upload methods)

public async Task<List<MediaFileDto>> ListFilesAsync(int levelId, int lessonId, string mediaType)
{
    // Define the folder prefix to search for in S3
    var prefix = $"levels/{levelId}/lessons/{lessonId}/{mediaType.ToLowerInvariant()}/";

    try
    {
        var request = new ListObjectsV2Request
        {
            BucketName = _awsSettings.MediaBucketName,
            Prefix = prefix
        };

        var response = await _s3Client.ListObjectsV2Async(request);

        // Map the S3 objects to our DTO
        var mediaFiles = response.S3Objects.Select(s3Obj => new MediaFileDto
        {
            Key = s3Obj.Key,
            Url = $"https://{_awsSettings.MediaBucketName}.s3.{_awsSettings.Region}.amazonaws.com/{s3Obj.Key}",
            // Extract the original file name, removing the GUID prefix
            FileName = Path.GetFileName(s3Obj.Key).Substring(37), // 36 chars for GUID + 1 for '_'
            Size = s3Obj.Size,
            LastModified = s3Obj.LastModified.ToUniversalTime()
        }).ToList();

        return mediaFiles;
    }
    catch (AmazonS3Exception ex)
    {
        // Log the exception
        throw new Exception($"Error listing files from S3: {ex.Message}", ex);
    }
}
```
**Key points about this method:**
*   It uses the `Prefix` parameter of the `ListObjectsV2Request` to tell S3 to only return files that are inside our specific folder (e.g., `levels/1/lessons/3/images/`).
*   It intelligently extracts the original file name by stripping off the `GUID_` prefix we added during upload.
*   It maps the raw S3 object data into our clean `MediaFileDto`.

---

### **Step 2: Update the `MediaController` (Web Layer)**

Now, we expose this new service method via a `GET` endpoint.

**File: `src/ICEDT_TamilApp.Web/Controllers/MediaController.cs` (Add this new method)**
```csharp
// ... (existing constructor and upload endpoints)

/// <summary>
/// Lists all files in a specific media folder for a given lesson.
/// </summary>
[HttpGet("list")]
[ProducesResponseType(typeof(List<MediaFileDto>), 200)]
public async Task<IActionResult> ListFiles(
    [FromQuery, Required] int levelId,
    [FromQuery, Required] int lessonId,
    [FromQuery, Required] string mediaType)
{
    var files = await _mediaService.ListFilesAsync(levelId, lessonId, mediaType);
    return Ok(files);
}
```
*Note: We use `[FromQuery]` because this is a `GET` request, and the parameters will be in the URL's query string (e.g., `/api/media/list?levelId=1&lessonId=3&mediaType=images`).*

---

### **Step 3: How to Use This in Your React Admin Panel**

Your backend is now ready. The React admin panel can use this new endpoint to build a "Media Browser". Here’s the conceptual workflow for your `ActivityEditorPage`:

1.  **Add a "Browse" Button:** Next to a field in your `ActivityForm` that needs a URL (e.g., an `imageUrl` field inside your `ContentJson` textarea), you would add a "Browse S3" button.

2.  **Open a Modal:** Clicking this button opens a new modal component, let's call it `<MediaBrowser />`.

3.  **The `<MediaBrowser />` Component:**
    *   When it opens, it makes an AJAX call to your new endpoint:
        ```javascript
        // Inside the MediaBrowser component
        const levelId = /* get from activity data */;
        const lessonId = /* get from activity data */;
        
        // Fetch images
        const images = await fetch(`/api/media/list?levelId=${levelId}&lessonId=${lessonId}&mediaType=images`);
        
        // Fetch audio
        const audioFiles = await fetch(`/api/media/list?levelId=${levelId}&lessonId=${lessonId}&mediaType=audio`);
        ```
    *   It receives the list of `MediaFileDto` objects.
    *   It displays these files in a user-friendly way (e.g., as a grid of image thumbnails or a list of audio file names with play buttons).
        ```html
        <!-- Inside the Media Browser UI -->
        <div class="image-grid">
            <img src="https://.../image1.png" alt="image1.png" />
            <img src="https://.../image2.jpg" alt="image2.jpg" />
        </div>
        ```
    *   Each image or file is clickable.

4.  **Selecting a File:**
    *   When the admin clicks on a file (e.g., `image2.jpg`), the `<MediaBrowser />` modal closes.
    *   It returns the selected file's `Url` back to the `ActivityForm`.
    *   The `ActivityForm`'s JavaScript then programmatically updates the `ContentJson` textarea, inserting the selected URL into the correct property.

**Example of the `ContentJson` update:**
```javascript
// Before selection
// { "question": "Which animal is this?" }

// After selecting an image from the Media Browser
// { "question": "Which animal is this?", "imageUrl": "https://.../image2.jpg" }
```

This creates a seamless and intuitive workflow for your content creators. They no longer need to manually copy and paste URLs. They can simply upload their media and then browse their organized S3 library to select the exact file they need for any given activity.
