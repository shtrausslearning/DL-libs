
## PyTorch Basic Setup

- We'll be utilising a simple dataset, containing two classes [0,1]
- Our problem will be a <code>binary</code> classification problem
- Train model on a <code>training</code> & <code>validation</code> datasets
- Display the <code>loss</code> history for the model on both datasets
- <code>inference</code> on part of the <code>validation</code> dataset

### Create Dataset

- <code>sklearn</code> make_moons creates a <code>two</code> class data structure (<code>samples</code> 400, <code>noise</code> 0.8)
- Training/Test split is utilised (<code>test_size</code> 0.2, <code>shuffle</code> True)
- <code>scaling</code> will be used before importing the data into the neural network

```python
from sklearn.datasets import make_moons
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split

X, y = make_moons(n_samples=400,
                  noise=0.8,
                  random_state=11)

X_train, X_val, y_train, y_val = train_test_split(X, y, 
                                                  test_size=0.2,
                                                  random_state=13,
                                                  shuffle=True)

sc = StandardScaler()
sc.fit(X_train)

X_train = sc.transform(X_train)
X_val = sc.transform(X_val)

```

### Visualise Dataset

- Having split the data into a <code>training</code> and <code>validation</code> datasets, let's visualise it

```python

import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots

fig = make_subplots(rows=1, cols=2,subplot_titles=['Train','Validation'])
itrace = px.scatter(X_train,
                    color=y_train,
                   )['data'][0]
trace = px.scatter(X_val,
                   color=y_val)['data'][0]

fig.add_trace(itrace, row=1, col=1)
fig.add_trace(trace, row=1, col=2)

fig.update_layout(template='plotly_white',
                  title='Train / Validation Data Splitting',
                  font=dict(family='sans-serif',size=12),
                  width=1200)

fig.update_traces({'marker_line_width':1.5, 
                   'marker_line_color':"black",
                   'marker_size':8,
                   'opacity':1.0,
#                    'marker':{'showscale':True,'reversescale':True, 'cmid':0, 'size':10},
                  })

fig.update_coloraxes(colorscale="tealgrn")
fig.update_layout(coloraxis_showscale=False)
fig.show()

```

### Data Preparation
- Create a tensor(s) from Numpy Arrays <code>X_train_tensor</code>, <code>y_train_tensor</code>, <code>X_val_tensor</code>, <code>y_val_tensor</code>
- Create a Dataset(s) (contains all data) <code>train_dataset</code>, <code>val_dataset</code>
- Create a DataLoader(s) (batch loading during training) <code>train_loader</code>, <code>val_loader</code>

```python

device = 'cuda' if torch.cuda.is_available() else 'cpu'

# Builds tensors from numpy arrays
x_train_tensor = torch.as_tensor(X_train).float()
y_train_tensor = torch.as_tensor(y_train[:,None]).float()
x_val_tensor = torch.as_tensor(X_val).float()
y_val_tensor = torch.as_tensor(y_val[:,None]).float()

# Builds dataset containing ALL data points
train_dataset = TensorDataset(x_train_tensor, y_train_tensor)
val_dataset = TensorDataset(x_val_tensor, y_val_tensor)

# Builds a loader of each set
batch_size = 10
train_loader = DataLoader(dataset=train_dataset,
                          batch_size=batch_size, 
                          shuffle=True)

val_loader = DataLoader(dataset=val_dataset,
                        batch_size=batch_size)
                        
```

What do <code>DataLoaders</code> contain:
- Batch Data (subset of the entire dataset)

```python
for x,y in train_loader:
    print(x,y)
    break
```

```
tensor([[-1.3243,  1.4676],
        [ 0.0957, -1.8608],
        [-0.1862,  2.0344],
        [-1.0247,  1.2672],
        [-0.6044,  0.7080]]) tensor([[0.],
        [1.],
        [0.],
        [0.],
        [0.]])
        
```

### PyTorch Training/Inference Operation Class

**Attributes:**
- <code>model</code> : model architecture nn.sequence
- <code>loss_fn</code> : loss function 
- <code>optimizer</code> : optimiser
- <code>train_loader</code> : storage space for training <code>DataLoader</code>
- <code>val_loader</code> : storage space for test <code>DataLoader</code>

**General Class Methods:**
- <code>to</code> : move data to <code>CPU</code> or <code>GPU</code>
- <code>set_loaders</code> : store data loaders in local variables
- <code>set_seed</code> : set seed 
- <code>train</code> : train model on both datasets
- <code>save_checkpoint</code> : save model, optmiser data needed to restart training
- <code>load_checkpoint</code> : load model,            "
- <code>predict</code> : predict on test data
- <code>plot_losses</code> : plot the training loss history 

```python

class pyRun(object):
    
    def __init__(self, model, loss_fn, optimiser):

        self.model = model
        self.loss_fn = loss_fn
        self.optimiser = optimiser
        self.device = 'cuda' if torch.cuda.is_available() else 'cpu'
        self.model.to(self.device)

        self.train_loader = None
        self.val_loader = None
        
        self.losses = []
        self.val_losses = []
        self.total_epochs = 0

        self.train_step = self._make_train_step() # store training step func
        self.val_step = self._make_val_step()  # store validatio step func
    
    def _make_train_step(self):

        def perform_train_step(x, y):

            self.model.train()             # set model to train mode
            yhat = self.model(x)           # compute model prediction (forward pass)
            loss = self.loss_fn(yhat, y)   # compute the loss 
            loss.backward() # compute gradients for both a,b parameters (backward pass)
            self.optimizer.step()      # update parameters using gradients and lr
            self.optimizer.zero_grad() # reset gradients

            return loss.item() # Returns the loss
            
        return perform_train_step # return function used in training loop
    
    def _make_val_step(self):
  
        def perform_val_step(x, y):
            self.model.eval() # eval mode
            yhat = self.model(x) # model prediction output (forward pass)
            loss = self.loss_fn(yhat, y) # compute loss 
            return loss.item()

        return perform_val_step
        
    def _mini_batch(self, validation=False):
        if validation:
            data_loader = self.val_loader
            step = self.val_step
        else:
            data_loader = self.train_loader
            step = self.train_step

        if data_loader is None:
            return None
            
        mini_batch_losses = []
        for x_batch, y_batch in data_loader:
            x_batch = x_batch.to(self.device)
            y_batch = y_batch.to(self.device)

            mini_batch_loss = step(x_batch, y_batch)
            mini_batch_losses.append(mini_batch_loss)

        loss = np.mean(mini_batch_losses)
        return loss
    
    def to(self, device):
        self.device = device
        self.model.to(self.device)
    
    def set_loaders(self, train_loader, val_loader=None):
        self.train_loader = train_loader
        self.val_loader = val_loader
    
    def set_seed(self, seed=42):
        torch.backends.cudnn.deterministic = True
        torch.backends.cudnn.benchmark = False    
        torch.manual_seed(seed)
        np.random.seed(seed)
    
    def train(self,
              n_epochs,
              seed=42):
        
        self.set_seed(seed)

        for epoch in range(n_epochs):

            self.total_epochs += 1
            loss = self._mini_batch(validation=False)
            self.losses.append(loss)
            
            with torch.no_grad():
                val_loss = self._mini_batch(validation=True) # eval w/ mini-batches
                self.val_losses.append(val_loss)

    def save_checkpoint(self, filename):
    
        checkpoint = {'epoch': self.total_epochs,
                      'model_state_dict': self.model.state_dict(),
                      'optimizer_state_dict': self.optimizer.state_dict(),
                      'loss': self.losses,
                      'val_loss': self.val_losses}

        torch.save(checkpoint, filename)

    def load_checkpoint(self, filename):

        checkpoint = torch.load(filename) # loads dict
        self.model.load_state_dict(checkpoint['model_state_dict'])
        self.optimizer.load_state_dict(checkpoint['optimizer_state_dict'])

        self.total_epochs = checkpoint['epoch']
        self.losses = checkpoint['loss']
        self.val_losses = checkpoint['val_loss']

        self.model.train() # when resuming training
        
    def predict(self, x):

        self.model.eval() # set to evaluation mode for predictions
        x_tensor = torch.as_tensor(x).float() # numpy & make float tensor
        y_hat_tensor = self.model(x_tensor.to(self.device)) # send input to device & use model for predictions
        self.model.train() # set it back to train mode

        return y_hat_tensor.detach().cpu().numpy() # detach it, brings it to CPU and back to np
    
    def plot_losses(self):
        
        fig = make_subplots(rows=1, cols=1,subplot_titles=None)
        itrace = go.Scatter(y=self.losses,marker_color='#283747',name='train')
        trace = go.Scatter(y=self.val_losses,marker_color='#C7C7C7',name='validation')

        fig.add_trace(itrace, row=1, col=1)
        fig.add_trace(trace, row=1, col=1)
        
        fig.update_layout(template='plotly_white',
                          title=f'Loss Function Convergence',
                          font=dict(family='sans-serif',size=12),
                          width=600)

        fig.update_traces(marker=dict(line=dict(width=0.5, color='white')),
                          opacity=0.75)
        fig.show()

```

### Set Neural Network 

- Set <code>PyTorch</code> compatible class (req. <code>init</code> & <code>forward</code>)

```Python
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.fc1 = nn.Linear(2,1000)
        self.fc2 = nn.Linear(1000,100)
        self.fc3 = nn.Linear(100,1)
        
    def forward(self,inputs):
        x = self.fc1(inputs)
#         x = self.fc1(inputs)
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x
        
model = Net() 
```

- Set prerequisites to <code>pyRun</code> : <code>loss function</code> & <code>optimiser</code>
- Model can either output <code>logits</code> or <code>probabilities</code> 
- Whatever the choice is, make sure you are using the appropriate loss function for the model
- Model <code>Net()</code> requires **WithLogits** loss function as <code>nn.Sigmoid()</code> is not added

```Python
optimizer = optim.SGD(model.parameters(), lr=0.01)
loss_fn = nn.BCEWithLogitsLoss()
```

### Train Neural Network
- Instantiate class <code>pyRun</code>

```Python
sbs = pyRun(model,      # model 
            loss_fn,    # loss function 
            optimizer)  # set optimiser
```

- Set <code>DataLoaders</code>

```Python
# Set DataLoaders
sbs.set_loaders(train_loader,
                val_loader)
```  
              
- Train model with <code>train</code> method

```Python
sbs.train(n_epochs=100)
```

- Visualise <code>loss</code> history

```Python
fig = sbs.plot_losses()
fig.show()
```

### Inference on Test Data

- Inference & show hit/misses of <code>classifier</code>

```Python

test_x, test_y = val_dataset[:]

model.eval()
logits = model(test_x.to(device))
probs = torch.sigmoid(logits)
probs

threshold = 0.5
confusion_matrix(some_y, (probs.cpu() >= threshold))
```
