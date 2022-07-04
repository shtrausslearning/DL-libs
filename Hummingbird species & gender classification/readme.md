![](https://i.imgur.com/icqghBp.png)

### 1 | Hummingbird monitoring

- The purpose of this project is to create an image <code>classifier</code> for hummingbird **species** and **genders** that visit feeders. 
- Such a classifier should be applicable to anywhere that hummingbirds migrate or breed, given additional datasets for those species.
- Hummingbird migration is otherwise reliant on individual bird watchers to see and report their observations. 
- If avid bird lovers setup a similar system, then conservation organizations would have better information on migratory and breeding patterns. 
- This knowledge can be used to determine if specific changes in the environment or ecology has positively or negatively impacted bird life.

### 2 | Reasons for turning to machine learning

- With the increased affordability of visual monitoring equipment, it has become practical for anyone to contribute to such a wonderful cause & help make each sighting more valid. 
- Often, due to <b>limitations of gear</b>, <b>poor photography/videography technique</b>, or **<span style='color:#B6DA32'>simply poor lighting conditions</span>**, it can be difficult for users, <b>let alone experts</b> to distinguish what specific bird was seen at the time of monitoring.
- In this study, we will focus our attention to a bird called the <b>hummingbird</b>. What is useful about this specie is that, despite being quite distinguishible in shape, <b>they have a variery of unique colouring</b>, which means that if images are of poor quality, it may be hard for humans or models to distinguish them.
- In the entire process of <b>expert identification</b> & dealing with various image related inconsistencies outlied previously, manual identification for monitoring can be quite labourous, so an automated system of identification can go a long way.
- In our quest to create an automated approach, we can be left with a collection or under or over exposed images that will create difficulties for the model to distinguish between different classes correctly. 

### 3 | Project goal 

- The ultimate goal is to have a classification system that can address such the above stated varieties in an image & correctly distinguish very similar bird species, it should be deployable at any feeder, which is important to the continued monitoring of hummingbird species and bird  migration patterns. 

### 4 | The dataset

- The dataset contains a main folder; <code>hummingbirds</code>, which contains image data split into <code>training</code>, <code>validation</code> & <code>test</code> sets, so the <code>train/test</code> split has already been done for us

```python
''' Folder Pathways'''
main_folder = '/kaggle/input/hummingbirds-at-my-feeders/'
train_folder = '/kaggle/input/hummingbirds-at-my-feeders/hummingbirds/train/'
val_folder = '/kaggle/input/hummingbirds-at-my-feeders/hummingbirds/valid/'
test_folder = '/kaggle/input/hummingbirds-at-my-feeders/hummingbirds/test/'
video_folder = '/kaggle/input/hummingbirds-at-my-feeders/video_test/'
```

```python

os.listdir(main_folder)

```

> ['video_test', 'All_images', 'hummingbirds']

```python
os.listdir(train_folder)
```

> ['Rufous_female', 'No_bird', 'Broadtailed_female', 'Broadtailed_male']

- Let's check how many images there are in each folder <code>Rufous female</code>, <code>No bird</code>, <code>Broadtailed female</code> & <code>Broadtailed male</code>
- We have a total of **4 classes**, so we'll be treating the problem as as <code>multiclass</code> classification problem

```python

class_types = len(os.listdir(train_folder))
print('Number of classes for Classification: ',class_types)
class_names = os.listdir(train_folder)
print(f'The class names are {class_names}\n')

print('Training dataset:')
for i in class_names:
    print(i + ':' + str(len(os.listdir(train_folder+i))))

print('\nValidation dataset:')
for i in class_names:
    print(i + ':' + str(len(os.listdir(val_folder+i))))
    
print('\nTest dataset:')
for i in class_names:
    print(i + ':' + str(len(os.listdir(test_folder+i))))
    
```

- The class distribution is even, the <code>training</code> data containing 100 samples per class & both <code>validation</code> and <code>test</code> data containing 20 samples per class

```
Number of classes for Classification:  4
The class names are ['Rufous_female', 'No_bird', 'Broadtailed_female', 'Broadtailed_male']

Training dataset:
Rufous_female:100
No_bird:100
Broadtailed_female:100
Broadtailed_male:100

Validation dataset:
Rufous_female:20
No_bird:20
Broadtailed_female:20
Broadtailed_male:20

Test dataset:
Rufous_female:20
No_bird:20
Broadtailed_female:20
Broadtailed_male:20
```

### 5 | Visualisation of Images

```python

''' Visualise Image Data '''
# Visualise a certain number of images in a folder using ImageGrid

def show_grid(image_list,nrows,ncols,label_list=None,
              show_labels=False,savename=None,
              figsize=(20,10),showaxis='off'):
    
    if type(image_list) is not list:
        if(image_list.shape[-1]==1):
            image_list = [image_list[i,:,:,0] for i in range(image_list.shape[0])]
        elif(image_list.shape[-1]==3):
            image_list = [image_list[i,:,:,:] for i in range(image_list.shape[0])]
    fig = plt.figure(None, figsize,frameon=False)
    grid = ImageGrid(fig, 111,  # similar to subplot(111)
                     nrows_ncols=(nrows, ncols),  # creates 2x2 grid of axes
                     axes_pad=0.3,  # pad between axes in inch.
                     share_all=True)
    
    for i in range(nrows*ncols):
        ax = grid[i]
        img = Image.open(image_list[i])
        ax.imshow(img,cmap='Greys_r')  # The AxesGrid object work as a list of axes.
        ax.axis(showaxis)
        if show_labels:
            ax.set_title(class_mapping[y_int[i]])
    if savename != None:
        plt.savefig(savename,bbox_inches='tight')
        
```
