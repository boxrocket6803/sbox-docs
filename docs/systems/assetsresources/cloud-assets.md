---
title: "Cloud Assets"
icon: "☁️"
created: 2024-12-17
updated: 2024-12-17
---

# Cloud Assets

There is a large selection of Assets (Textures, Models, Sounds, ect) available to use on sbox.game, and you can use them without needing to think about downloading the files/mounting the content/ect.

# Cloud Browser

The Cloud Browser allows you to instantly drag any Asset directly into your scene to be used in your game right away. It will download all necessary assets behind-the-scenes and cache them so you don't need to worry about downloading them each time. This is the quickest and easiest way to reference Cloud Assets

![Using the Cloud Browser to add different Models/Materials to a Scene 1882x896](./images/using-the-cloud-browser-to-add-different-models-materials-to.mp4)

# Referencing in Code via Properties

If you expose your variables as Properties on a Component or GameResource, you can use the Cloud Browser to reference an Asset and it will be treated just as a local Asset would.

```csharp
public Model MyModel { get; set; } // Can reference a Local Model OR a Cloud Model
```

# Referencing in Code via Ident

If you know the ident/url of a cloud asset, you can use the `Cloud.*` functions to directly reference them.

```csharp
MyModel = Cloud.Model( "facepunch.w_usp" ); // Dot notation
MyModel = Cloud.Model( "facepunch/w_usp" ); // Slash notation
MyModel = Cloud.Model( "https://sbox.game/facepunch/w_usp" ); // The direct URL to the Asset
```

These assets are downloaded during code compilation, so everything is treated like a local model behind the scenes. These assets will be shipped as part of your package, meaning if a model is changed later on, it won't update until you re-publish your package.

# Mounting Cloud Assets at Runtime

Sometimes you might need to download and mount certain resources from sbox.game at runtime. Maybe you're making a GMod style game with spawnable props, or maybe you're procedurally generating something.

```csharp
static async Task<Model> DownloadModel( string packageIdent )
{
  var package = await Package.Fetch( packageIdent, false );
  if( package == null || package.Revision == null )
  {
    // Package not found
    return;
  }
  // If the package was found, mount it (download the content)
  await package.MountAsync();
  
  // Get the path to the primary asset (vmdl for Model, vsnd for Sound, ect.)
  var primaryAsset = package.GetMeta( "PrimaryAsset", "" );
  return Model.Load( primaryAsset );
}
```
