### Why
我们是做 saas 系统的，这意味着我们必须从不同客户的需求中提炼出通用逻辑，并针对不同客户，提供完善、标准的服务去覆盖他们的需求，但是在投入市场的前期，我们的软件通常不完善，不具备完整的能力。客户会根据他们的需求提出一些定制化的开发工作，这其中通用部分会被沉淀下来成为标准产品，而其他的会在我们的标准化分支上做定制化开发。

在之前，我们定制化开发的代码都会停留在指定的标准分支上，而不影响标准代码，但是伴随着客户的增多，我们需要为客户升级版本（使用新的标准分值代码覆盖老的代码），这时候我们发现各种组件、业务的代码都需要合并到新的标准分支上才行，这很难管理。

### What
#### 组件
组件都使用同一个源，仅通过不同的 version 来管理使用的版本和逻辑。
#### 业务
针对特定的业务需求，会在原有代码上手写代码实现逻辑。